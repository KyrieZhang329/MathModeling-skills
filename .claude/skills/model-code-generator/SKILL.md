---
name: model-code-generator
description: Generate executable Python or MATLAB modeling code from a validated method plan.
license: MIT
---

# Purpose

Generate clear, executable, and reproducible modeling code from a validated method plan and cleaned data.

This skill converts the selected baseline, main, and improved models into runnable Python or MATLAB scripts. It should produce code that reads cleaned data, implements the planned methods, saves intermediate and final outputs, and creates model-related figures when required by the method plan.

This skill does not choose the modeling route, clean raw data, perform final code review, run final QA, or write paper sections.

# When to use

Use this skill:

- After `method-selector` has produced a validated method plan.
- After `data-auditor-cleaner` has confirmed that the required data is ready or ready with documented warnings.
- When executable scripts are needed for baseline, main, or improved models.
- When result tables, intermediate outputs, and model figures must be generated reproducibly.
- Before `code-reviewer`.

# Preconditions

The following should already exist or be provided:

- A validated problem parse.
- A validated problem classification artifact.
- A validated method plan.
- Cleaned data or a clear statement that the model does not require tabular data.
- Data audit report and field mapping, if data is used.
- Required inputs, expected outputs, validation needs, and robustness needs from the method plan.
- Preferred implementation language, if specified.

If the method plan is missing, hand back to `method-selector`.

If cleaned data is required but missing, hand back to `data-auditor-cleaner`.

# Inputs

Use or request:

- `workspace/problem/problem_parse.json`, if available.
- `workspace/problem/problem_classification.json`, if available.
- `workspace/problem/method_plan.json`, if available.
- `workspace/results/data_report.md`, if available.
- Cleaned data under `workspace/data_clean/`.
- Data field mapping from `data-auditor-cleaner`.
- Method plan sections for baseline, main model, improved model, validation, expected results, and expected figures.
- Preferred language: Python by default unless MATLAB is requested.
- Contest constraints, such as reproducibility, runtime limits, or allowed dependencies.

# Workflow

1. Read the validated method plan.
   - Identify each subquestion.
   - Identify baseline, main model, and improved model requirements.
   - Identify expected output tables and figures.
   - Do not change the selected modeling route unless it is impossible to implement.

2. Check data readiness.
   - Confirm cleaned data paths exist or are specified.
   - Confirm required fields are available.
   - Confirm field names used in code match the field mapping.
   - Do not read from or modify `workspace/data_raw/` unless only inspecting metadata is explicitly required.

3. Plan code structure.
   - Prefer one script per subquestion when tasks are separable.
   - Use clear function boundaries only when they reduce repetition or improve auditability.
   - Avoid unnecessary abstractions.
   - Keep the code easy for `code-reviewer` and teammates to inspect.

4. Generate baseline code first.
   - Implement the simplest valid baseline.
   - Save baseline outputs.
   - Make baseline outputs comparable with the main model.

5. Generate main model code.
   - Implement the selected method from the method plan.
   - Use variable names aligned with the method plan and expected paper notation where practical.
   - Save intermediate outputs needed for explanation.

6. Generate improved model code only if justified.
   - Implement optional improvements only if the method plan marks them as feasible.
   - Clearly separate optional improvement code from required baseline and main model code.
   - Do not add advanced methods only for appearance.

7. Add reproducibility and artifact saving.
   - Fix random seeds when randomness is used.
   - Save result tables under `workspace/results/`.
   - Save figures under `workspace/figures/`.
   - Save scripts under `workspace/scripts/`.
   - Use relative paths where possible.

8. Add minimal comments.
   - Explain key modeling steps.
   - Explain non-obvious formulas.
   - Explain important assumptions.
   - Do not over-comment trivial syntax.

9. Produce a code generation summary.
   - List generated scripts.
   - List expected outputs.
   - List how to run the scripts.
   - List known implementation risks.
   - Recommend `code-reviewer` as the next skill.

# Outputs

Produce code and supporting artifacts such as:

- `workspace/scripts/model_q1.py` or `workspace/scripts/model_q1.m`
- `workspace/scripts/model_q2.py` or equivalent
- `workspace/results/q1_baseline_results.csv`
- `workspace/results/q1_main_results.csv`
- `workspace/results/q1_intermediate_outputs.csv`
- `workspace/figures/q1_*.png`
- run instructions
- implementation summary
- known risks
- recommended next skill

# Output format

Prefer this JSON-compatible summary:

```json
{
  "code_generation_summary": {
    "language": "python",
    "generated_scripts": [
      "workspace/scripts/model_q1.py"
    ],
    "data_inputs": [
      "workspace/data_clean/clean_data.csv"
    ],
    "result_outputs": [
      "workspace/results/q1_baseline_results.csv",
      "workspace/results/q1_main_results.csv"
    ],
    "figure_outputs": [
      "workspace/figures/q1_ranking_plot.png"
    ],
    "run_instructions": [
      "python workspace/scripts/model_q1.py"
    ],
    "known_risks": [
      "Column units must be confirmed before final interpretation."
    ],
    "recommended_next_skill": "code-reviewer"
  }
}
```

If a JSON block is too rigid for the situation, use a concise Markdown report with the same fields.

# Code generation rules

- Generate code only from a validated method plan.
- Prefer Python unless the user explicitly requests MATLAB.
- Use minimal standard dependencies.
- For Python, prefer common scientific libraries such as `pandas`, `numpy`, `scipy`, `scikit-learn`, and `matplotlib` when appropriate.
- Do not introduce heavy or obscure dependencies unless the method plan requires them.
- Keep imports minimal.
- Use relative paths when practical.
- Fix random seeds when using randomness.
- Separate baseline outputs from main model outputs.
- Save all important numeric results to files.
- Save figures to files rather than relying only on interactive display.
- Include enough comments to connect code to the method plan.
- Avoid single-use over-abstractions.
- Avoid hidden global state.
- Avoid hard-coded absolute local paths.
- Avoid changing the modeling route inside the code.
- Avoid silently dropping rows or columns.
- Avoid leaking future information into prediction features.
- Avoid using raw data files as write targets.

# Artifact rules

- Read cleaned data from `workspace/data_clean/`.
- Do not modify files under `workspace/data_raw/`.
- Save generated scripts under `workspace/scripts/`.
- Save numeric outputs under `workspace/results/`.
- Save generated figures under `workspace/figures/`.
- Preserve file names that make subquestion and model role clear.
- Use names such as:
  - `model_q1.py`
  - `model_q1_baseline.py`
  - `model_q1_main.py`
  - `q1_baseline_results.csv`
  - `q1_main_results.csv`
  - `q1_model_summary.json`
- If a file may overwrite an existing artifact, mention it before doing so or choose a clearly versioned name.

# Rules

- Do not select or change the model plan except to report infeasibility.
- Do not clean raw data.
- Do not fabricate data, labels, metrics, or results.
- Do not write final paper text.
- Do not perform final QA.
- Do not claim that code is correct before review.
- Do not hide implementation assumptions.
- Do not use advanced models without explicit method-plan support.
- Do not create decorative figures unrelated to expected artifacts.
- Do not ignore validation needs listed in the method plan.
- Mark incomplete or risky implementation parts explicitly.
- If code cannot be generated safely, stop and report blockers.

# Verification

Before handing off, verify:

- Every generated script maps to a specific subquestion and method-plan item.
- Baseline code exists for every main model unless explicitly impossible.
- Required cleaned data paths are used.
- No raw data file is overwritten.
- Random seeds are fixed where needed.
- Outputs are saved under the correct workspace directories.
- Result tables have clear file names.
- Figures, if generated, support expected model outputs.
- Run instructions are provided.
- Known risks and assumptions are listed.
- The next skill is `code-reviewer`.

# Failure modes

Stop and report a blocker if:

- No validated method plan exists.
- Cleaned data is required but unavailable.
- Required fields are missing from cleaned data.
- The selected method cannot be implemented with available data.
- The required model needs dependencies that are unavailable or inappropriate for contest use.
- The expected output is not defined.
- Generating code would require inventing data, labels, parameters, or metrics.
- The user asks for final paper claims or final submission before code review and results exist.

# Stop conditions

This skill must stop instead of guessing when:

- The method plan is ambiguous enough that different implementations would produce different meanings.
- Required model variables cannot be mapped to cleaned data fields.
- The model depends on unavailable attachments or forbidden external data.
- The code would need to silently choose important parameters not specified by the method plan.
- Continuing would overwrite validated artifacts without permission.

When stopping, output:

- the blocker
- why it matters
- affected subquestion or model
- safe partial implementation if any
- missing information needed
- recommended next action

# Handoff

After generating code and implementation summary, hand off to:

`code-reviewer`

The handoff should include:

- generated script paths
- data input paths
- expected result output paths
- expected figure output paths
- run instructions
- method-plan references
- known risks and assumptions
- incomplete implementation notes if any

Do not hand off directly to `paper-section-writer` or `quality-assurance-auditor`.

# Examples

## Example 1: Generate evaluation model code

Input state:

- Q1 method plan includes equal-weight baseline and entropy-weight TOPSIS.
- Cleaned data exists at `workspace/data_clean/indicator_data.csv`.

Output:

```json
{
  "code_generation_summary": {
    "language": "python",
    "generated_scripts": [
      "workspace/scripts/model_q1.py"
    ],
    "data_inputs": [
      "workspace/data_clean/indicator_data.csv"
    ],
    "result_outputs": [
      "workspace/results/q1_equal_weight_results.csv",
      "workspace/results/q1_entropy_topsis_results.csv"
    ],
    "figure_outputs": [
      "workspace/figures/q1_ranking_comparison.png"
    ],
    "run_instructions": [
      "python workspace/scripts/model_q1.py"
    ],
    "known_risks": [
      "Indicator directions must match the method plan before interpretation."
    ],
    "recommended_next_skill": "code-reviewer"
  }
}
```

## Example 2: Blocked by missing cleaned data

Input state:

- Method plan exists.
- Raw data exists.
- No cleaned data exists.

Output:

```json
{
  "blocked_items": [
    "Cleaned data is required before model code generation."
  ],
  "recommended_next_skill": "data-auditor-cleaner",
  "recommended_next_action": "Audit and clean the raw data, then provide cleaned data paths and field mapping."
}
```

## Example 3: Blocked by method-data mismatch

Input state:

- Method plan requires a labeled target variable.
- Cleaned data has features but no target label.

Output:

```json
{
  "blocked_items": [
    "The selected supervised model requires a target label, but no label field exists in the cleaned data."
  ],
  "affected_model": "Q2 main model",
  "recommended_next_skill": "method-selector",
  "recommended_next_action": "Revise the method plan to use an unsupervised, descriptive, or proxy-based approach, or provide valid labeled data."
}
```

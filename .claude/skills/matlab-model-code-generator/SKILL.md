---
name: matlab-model-code-generator
description: Generate executable MATLAB or Beita Tianyuan compatible modeling code from a validated method plan and cleaned data.
license: MIT
---

# Purpose

Generate MATLAB / 北太天元 compatible modeling code from a validated method plan and cleaned data.

This skill implements the approved baseline, main model, and optional improved model as `.m` scripts. It should produce code that reads cleaned data, runs the planned model, saves result tables, saves figures, and produces portable artifacts for downstream robustness checks and paper writing.

This skill does not choose the model, clean raw data, review code, run final QA, or write paper sections.

# When to use

Use this skill:

- After `model-code-analyzer` hands off to `matlab-model-code-generator`.
- When `implementation.target = matlab`.
- When the contest requires MATLAB / 北太天元 compatible scripts.
- When the method plan and cleaned data are ready.
- When executable `.m` scripts are needed for baseline, main, or improved models.

# Preconditions

The following should already exist or be provided:

- A validated problem parse.
- A validated problem classification artifact.
- A validated method plan.
- `implementation.target = matlab`.
- Runtime notes if MATLAB / 北太天元 compatibility is required.
- Cleaned data under `workspace/data/data_clean/`, unless the model does not need tabular data.
- Data audit report and field mapping if data is used.
- Expected result files and figure files from the method plan.

If the method plan is missing, hand back to `method-selector`.

If cleaned data is required but missing, hand back to `data-auditor-cleaner`.

If `implementation.target` is not `matlab`, hand back to `model-code-analyzer`.

# Inputs

Use or request:

- `workspace/problem/method-selector/method_plan.json`
- `workspace/code/model-code-analyzer.md`, if available
- `workspace/data/data_report.md`, if available
- cleaned data under `workspace/data/data_clean/`
- field mapping from the data audit stage
- required model inputs
- expected result outputs
- expected figure outputs
- runtime notes such as:
  - `beita-tianyuan-compatible`
  - `avoid-heavy-toolboxes`
  - `prefer-basic-matrix-and-table-operations`
  - `avoid-live-scripts`
  - `avoid-app-designer`
  - `save-portable-artifacts`

# Workflow

1. Read the method plan.
   - Identify each subquestion.
   - Identify baseline, main model, and optional improved model.
   - Identify required inputs, expected outputs, validation needs, and robustness needs.
   - Do not change the selected modeling route.

2. Confirm MATLAB target.
   - Confirm `implementation.target = matlab`.
   - Preserve all runtime notes.
   - If 北太天元 compatibility is required, use conservative MATLAB-compatible syntax.

3. Check data readiness.
   - Use cleaned data from `workspace/data/data_clean/`.
   - Do not read from `workspace/data/data_raw/` unless only inspecting metadata is explicitly requested.
   - Do not overwrite raw data.
   - Confirm required fields match the data audit report.

4. Plan script structure.
   - Save MATLAB code under `workspace/code/matlab/`.
   - Use one folder per subquestion such as `workspace/code/matlab/Q1/` or `workspace/code/matlab/Q2/`.
   - Prefer plain `.m` scripts over Live Scripts.
   - Prefer simple script files for contest usability.
   - Use functions only when they make the code clearer; if using functions, ensure function names match file names.
   - Keep one script per subquestion when tasks are separable.
   - Use a `run_all.m` script only if it clearly improves execution order.

5. Generate baseline code.
   - Implement the baseline first.
   - Save baseline outputs separately.
   - Make baseline outputs comparable with the main model.

6. Generate main model code.
   - Implement the approved main model.
   - Use variable names that match the method plan where practical.
   - Save intermediate values needed for explanation or robustness checks.

7. Generate optional improved model code only if justified.
   - Add optional improvements only when the method plan marks them as feasible.
   - Keep optional improvement code clearly separated.
   - Do not add advanced methods for appearance.

8. Save artifacts.
   - Save result tables under `workspace/results/`.
   - Save figures under `workspace/figures/`.
   - Save `.mat` files when useful for later MATLAB review.
   - Prefer portable outputs such as `.csv`, `.mat`, and `.png`.
   - Do not rely only on command window output.

9. Add run instructions.
   - State the script order.
   - State expected input files.
   - State expected output files.
   - State known MATLAB / 北太天元 compatibility risks.

10. Hand off to `code-reviewer`.
   - The router should then route `.m` scripts to `matlab-code-reviewer`.

# Outputs

Produce MATLAB implementation artifacts such as:

- `workspace/code/matlab/Q1/q1_baseline.m`
- `workspace/code/matlab/Q1/q1_main.m`
- `workspace/code/matlab/Q1/q1_improved.m`
- `workspace/code/matlab/run_all.m`
- `workspace/code/matlab/code_generation_summary.md`
- `workspace/results/q1_baseline_results.csv`
- `workspace/results/q1_main_results.csv`
- `workspace/results/q1_model_summary.mat`
- `workspace/figures/q1_ranking.png`
- run instructions
- implementation summary
- known compatibility risks
- recommended next skill

# Output format

Prefer this JSON-compatible summary:

```json
{
  "matlab_code_generation_summary": {
    "implementation_target": "matlab",
    "runtime_notes": [
      "beita-tianyuan-compatible",
      "avoid-heavy-toolboxes"
    ],
    "generated_scripts": [
      "workspace/code/matlab/Q1/q1_baseline.m",
      "workspace/code/matlab/Q1/q1_main.m"
    ],
    "data_inputs": [
      "workspace/data/data_clean/clean_data.csv"
    ],
    "result_outputs": [
      "workspace/results/q1_baseline_results.csv",
      "workspace/results/q1_main_results.csv"
    ],
    "figure_outputs": [
      "workspace/figures/q1_ranking.png"
    ],
    "run_instructions": [
      "Run workspace/code/matlab/Q1/q1_baseline.m",
      "Run workspace/code/matlab/Q1/q1_main.m"
    ],
    "markdown_summary": "workspace/code/matlab/code_generation_summary.md",
    "known_risks": [
      "Compatibility with 北太天元 should be checked if toolbox-specific functions are introduced."
    ],
    "recommended_next_skill": "code-reviewer"
  }
}
```

If JSON is too rigid, use a concise Markdown report with the same fields.

# MATLAB / 北太天元 coding rules

Use conservative MATLAB-compatible code.

Prefer:

- plain `.m` scripts
- basic matrix operations
- `readtable`
- `writetable`
- `readmatrix`
- `writematrix`
- `save`
- `load`
- `plot`, `bar`, `scatter`, `histogram`
- `rng(seed)`
- clear relative paths
- explicit result saving
- simple loops and vectorized operations when readable

Avoid unless explicitly approved:

- Live Scripts
- App Designer
- GUI code
- Simulink dependencies
- parallel toolbox
- deep learning toolbox
- symbolic toolbox
- heavy toolbox-specific functions
- version-specific MATLAB features
- third-party packages
- absolute local paths
- scripts that only display results without saving them

For 北太天元 compatibility:

- Prefer simple MATLAB syntax.
- Avoid relying on advanced toolbox functions.
- Save portable artifacts.
- If uncertain whether a function is supported, choose a simpler implementation.
- Keep comments clear enough for manual translation or debugging.

# Path rules

Use relative paths where practical.

Recommended paths:

```text
workspace/data/data_clean/
workspace/code/matlab/
workspace/results/
workspace/figures/
```

Avoid hard-coded absolute paths such as:

```text
/Users/...
C:\Users\...
D:\...
```

If path setup is needed, define paths near the top of the script:

```matlab
dataDir = fullfile('workspace', 'data_clean');
resultDir = fullfile('workspace', 'results');
figureDir = fullfile('workspace', 'figures');
```

Create output directories if needed:

```matlab
if ~exist(resultDir, 'dir')
    mkdir(resultDir);
end

if ~exist(figureDir, 'dir')
    mkdir(figureDir);
end
```

# Artifact rules

- Read cleaned data from `workspace/data/data_clean/`.
- Do not modify files under `workspace/data/data_raw/`.
- Save MATLAB scripts under `workspace/code/matlab/`.
- Save numeric outputs under `workspace/results/`.
- Save figures under `workspace/figures/`.
- Use clear file names such as:
  - `q1_baseline.m`
  - `q1_main.m`
  - `q1_improved.m`
  - `q1_baseline_results.csv`
  - `q1_main_results.csv`
  - `q1_model_summary.mat`
  - `q1_ranking.png`
- Separate baseline outputs from main model outputs.
- Save important intermediate results if later paper explanation or robustness checks require them.

# Randomness rules

If randomness is used:

- Set a fixed seed with `rng(seed)`.
- Record the seed in comments and summary.
- Avoid hidden random initialization.
- Save outputs from each stochastic run if robustness checks require repeated trials.

Example:

```matlab
rng(2026);
```

# Commenting rules

Use comments to explain:

- data inputs
- key variables
- model steps
- non-obvious formulas
- output files
- assumptions inherited from the method plan

Avoid over-commenting trivial syntax.

# Rules

- Do not select or change the model.
- Do not clean raw data.
- Do not fabricate data, labels, parameters, metrics, or results.
- Do not write paper text.
- Do not perform final QA.
- Do not use Python-only output formats.
- Do not depend on heavy MATLAB toolboxes unless the method plan explicitly allows it.
- Do not hide assumptions inside code.
- Do not silently drop rows or columns.
- Do not overwrite raw data.
- Do not claim the code is reviewed or correct before `matlab-code-reviewer`.

# Verification

Before handoff, verify:

- `implementation.target = matlab`.
- Every generated script maps to a subquestion and method-plan item.
- Baseline code exists for every main model unless explicitly impossible.
- Scripts are saved under `workspace/code/matlab/`.
- Cleaned data paths are used.
- No raw data file is overwritten.
- Random seeds are fixed where needed.
- Results are saved under `workspace/results/`.
- Figures are saved under `workspace/figures/`.
- Outputs are portable enough for downstream checks.
- MATLAB / 北太天元 compatibility risks are listed.
- The next skill is `code-reviewer`.

# Failure modes

Stop and report a blocker if:

- The method plan is missing.
- The method plan is not validated.
- `implementation.target` is not `matlab`.
- Cleaned data is required but unavailable.
- Required data fields are missing.
- The selected method cannot be implemented without unsupported toolbox functions.
- The expected output is not defined.
- Generating code would require inventing data, labels, parameters, or metrics.
- MATLAB / 北太天元 compatibility requirements conflict with the selected method.

# Stop conditions

This skill must stop instead of guessing when:

- Model variables cannot be mapped to cleaned data fields.
- Important parameters are unspecified and cannot be safely defaulted.
- The method requires a toolbox or function that may be unavailable under 北太天元 and no simpler alternative is specified.
- The code would need to overwrite raw data.
- A script would produce results that cannot be traced to the method plan.
- Continuing would change the mathematical meaning of the model.

When stopping, output:

- blocker
- why it matters
- affected subquestion or model
- missing information or artifact
- safe partial implementation if any
- recommended next skill
- next action

# Handoff

After generating MATLAB scripts, hand off to:

```
code-reviewer
```

The handoff should include:

- generated `.m` script paths
- method plan path
- cleaned data paths
- field mapping
- expected result paths
- expected figure paths
- runtime notes
- MATLAB / 北太天元 compatibility requirements
- run instructions
- known risks and assumptions

`code-reviewer` should then route the scripts to:

```
matlab-code-reviewer
```

Do not hand off directly to `robustness-checker`.

# Examples

## Example 1: Generate MATLAB evaluation model code

Input state:

- Q1 uses equal-weight baseline and entropy-weight TOPSIS.
- Cleaned indicator data exists.
- `implementation.target = matlab`.
- Runtime notes include `beita-tianyuan-compatible`.

Output:

```json
{
  "matlab_code_generation_summary": {
    "implementation_target": "matlab",
    "generated_scripts": [
      "workspace/code/matlab/Q1/q1_baseline.m",
      "workspace/code/matlab/Q1/q1_entropy_topsis.m"
    ],
    "data_inputs": [
      "workspace/data/data_clean/indicator_data.csv"
    ],
    "result_outputs": [
      "workspace/results/q1_equal_weight_results.csv",
      "workspace/results/q1_entropy_topsis_results.csv"
    ],
    "figure_outputs": [
      "workspace/figures/q1_ranking_comparison.png"
    ],
    "run_instructions": [
      "Run workspace/code/matlab/Q1/q1_baseline.m",
      "Run workspace/code/matlab/Q1/q1_entropy_topsis.m"
    ],
    "known_risks": [
      "Indicator direction must match the method plan before interpretation."
    ],
    "recommended_next_skill": "code-reviewer"
  }
}
```

## Example 2: Blocked by missing cleaned data

Input state:

- Method plan exists.
- `implementation.target = matlab`.
- Raw data exists.
- Cleaned data does not exist.

Output:

```json
{
  "blocked_items": [
    "Cleaned data is required before MATLAB code generation."
  ],
  "recommended_next_skill": "data-auditor-cleaner",
  "recommended_next_action": "Audit and clean the raw data, then provide cleaned data paths and field mapping."
}
```

## Example 3: Blocked by unsupported implementation target

Input state:

- Method plan says `implementation.target = python`.
- This skill was invoked directly.

Output:

```json
{
  "blocked_items": [
    "matlab-model-code-generator was invoked, but implementation.target is python."
  ],
  "recommended_next_skill": "model-code-analyzer",
  "recommended_next_action": "Route the method plan through model-code-analyzer."
}
```

## Example 4: Blocked by heavy toolbox dependency

Input state:

- Method plan requires a toolbox-specific optimization function.
- Runtime notes require 北太天元 compatibility and avoiding heavy toolboxes.
- No simpler alternative is specified.

Output:

```json
{
  "blocked_items": [
    "The selected implementation appears to require a toolbox-specific optimization function, which conflicts with the 北太天元 compatibility constraint."
  ],
  "affected_model": "Q3 main model",
  "recommended_next_skill": "method-selector",
  "recommended_next_action": "Revise the method plan to use a basic matrix-based or explicitly supported solver approach."
}
```

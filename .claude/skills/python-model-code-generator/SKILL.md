---
name: python-model-code-generator
description: Generate executable Python modeling code from a validated method plan and cleaned data.
license: MIT
---

# Purpose

Generate Python modeling code from a validated method plan and cleaned data.

This skill implements the approved baseline, main model, and optional improved model as Python scripts. It should produce code that reads cleaned data, runs the planned model, saves result tables, saves figures, and produces portable artifacts for downstream robustness checks and paper writing.

This skill does not choose the model, clean raw data, review code, run final QA, or write paper sections.

# When to use

Use this skill:

- After `model-code-analyzer` hands off to `python-model-code-generator`.
- When `implementation.target = python`.
- When Python is allowed by the contest or requested by the user.
- When the method plan and cleaned data are ready.
- When executable `.py` scripts are needed for baseline, main, or improved models.
- When Python is used for data preprocessing, prototyping, validation, or final modeling.

# Preconditions

The following should already exist or be provided:

- A validated problem parse.
- A validated problem classification artifact.
- A validated method plan.
- `implementation.target = python`.
- Cleaned data under `workspace/data/data_clean/`, unless the model does not need tabular data.
- Data audit report and field mapping if data is used.
- Expected result files and figure files from the method plan.
- Runtime notes if specific dependency or portability constraints exist.

If the method plan is missing, hand back to `method-selector`.

If cleaned data is required but missing, hand back to `data-auditor-cleaner`.

If `implementation.target` is not `python`, hand back to `model-code-analyzer`.

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
  - `minimal-dependencies`
  - `portable-artifacts`
  - `avoid-notebook-only-code`
  - `save-csv-json-png`
  - `fixed-random-seed`

# Workflow

1. Read the method plan.
   - Identify each subquestion.
   - Identify baseline, main model, and optional improved model.
   - Identify required inputs, expected outputs, validation needs, and robustness needs.
   - Do not change the selected modeling route.

2. Confirm Python target.
   - Confirm `implementation.target = python`.
   - Preserve all runtime notes.
   - Use a minimal, common scientific Python stack.

3. Check data readiness.
   - Use cleaned data from `workspace/data/data_clean/`.
   - Do not read from `workspace/data/data_raw/` unless only inspecting metadata is explicitly requested.
   - Do not overwrite raw data.
   - Confirm required fields match the data audit report.

4. Plan script structure.
   - Save Python code under `workspace/code/python/`.
   - Use one folder per subquestion such as `workspace/code/python/Q1/` or `workspace/code/python/Q2/`.
   - Prefer plain `.py` scripts over notebook-only workflows.
   - Keep scripts runnable from the project root.
   - Keep one script per subquestion when tasks are separable.
   - Use a `run_all.py` script only if it clearly improves execution order.
   - Use small helper functions when they improve clarity.

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
   - Prefer portable outputs such as `.csv`, `.json`, and `.png`.
   - Do not rely only on console output.
   - Avoid Python-only binary outputs unless explicitly useful and documented.

9. Add run instructions.
   - State the script order.
   - State expected input files.
   - State expected output files.
   - State required dependencies.
   - State known portability risks.

10. Hand off to `code-reviewer`.
   - The router should then route `.py` scripts to `python-code-reviewer`.

# Outputs

Produce Python implementation artifacts such as:

- `workspace/code/python/Q1/q1_baseline.py`
- `workspace/code/python/Q1/q1_main.py`
- `workspace/code/python/Q1/q1_improved.py`
- `workspace/code/python/run_all.py`
- `workspace/code/python/code_generation_summary.md`
- `workspace/results/q1_baseline_results.csv`
- `workspace/results/q1_main_results.csv`
- `workspace/results/q1_model_summary.json`
- `workspace/figures/q1_ranking.png`
- run instructions
- implementation summary
- known dependency or portability risks
- recommended next skill

# Output format

Prefer this JSON-compatible summary:

```json
{
  "python_code_generation_summary": {
    "implementation_target": "python",
    "runtime_notes": [
      "minimal-dependencies",
      "portable-artifacts"
    ],
    "generated_scripts": [
      "workspace/code/python/Q1/q1_baseline.py",
      "workspace/code/python/Q1/q1_main.py"
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
      "python workspace/code/python/Q1/q1_baseline.py",
      "python workspace/code/python/Q1/q1_main.py"
    ],
    "markdown_summary": "workspace/code/python/code_generation_summary.md",
    "dependencies": [
      "numpy",
      "pandas",
      "matplotlib"
    ],
    "known_risks": [
      "Column names must match the data audit report before interpretation."
    ],
    "recommended_next_skill": "code-reviewer"
  }
}
```

If JSON is too rigid, use a concise Markdown report with the same fields.

# Python coding rules

Use clear, minimal, reviewable Python code.

Prefer:

- plain `.py` scripts
- `pathlib.Path`
- `numpy`
- `pandas`
- `scipy` when needed
- `scikit-learn` when justified
- `matplotlib`
- relative paths
- explicit result saving
- fixed random seeds
- portable outputs such as `.csv`, `.json`, and `.png`

Avoid unless explicitly approved:

- notebook-only code
- hard-coded local absolute paths
- hidden data cleaning inside modeling scripts
- heavy deep learning frameworks
- unnecessary third-party packages
- Python-only output formats as the only result artifact
- code that only prints results without saving them
- complex abstractions that make contest debugging harder

Common accepted dependencies:

```text
numpy
pandas
scipy
scikit-learn
matplotlib
```

Avoid heavy dependencies unless the method plan explicitly justifies them:

```text
torch
tensorflow
xgboost
lightgbm
large geospatial packages
specialized optimization packages
```

# Path rules

Use relative paths where practical.

Recommended paths:

```text
workspace/data/data_clean/
workspace/code/python/
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

```python
from pathlib import Path

ROOT = Path(".")
DATA_DIR = ROOT / "workspace" / "data" / "data_clean"
RESULT_DIR = ROOT / "workspace" / "results"
FIGURE_DIR = ROOT / "workspace" / "figures"

RESULT_DIR.mkdir(parents=True, exist_ok=True)
FIGURE_DIR.mkdir(parents=True, exist_ok=True)
```

# Artifact rules

- Read cleaned data from `workspace/data/data_clean/`.
- Do not modify files under `workspace/data/data_raw/`.
- Save Python scripts under `workspace/code/python/`.
- Save numeric outputs under `workspace/results/`.
- Save figures under `workspace/figures/`.
- Use clear file names such as:
  - `q1_baseline.py`
  - `q1_main.py`
  - `q1_improved.py`
  - `q1_baseline_results.csv`
  - `q1_main_results.csv`
  - `q1_model_summary.json`
  - `q1_ranking.png`
- Separate baseline outputs from main model outputs.
- Save important intermediate results if later paper explanation or robustness checks require them.

# Randomness rules

If randomness is used:

- Set fixed seeds.
- Record the seed in comments and summary.
- Avoid hidden random initialization.
- Save outputs from each stochastic run if robustness checks require repeated trials.

Example:

```python
import random
import numpy as np

SEED = 2026
random.seed(SEED)
np.random.seed(SEED)
```

If using scikit-learn models with randomness, pass `random_state=SEED` where supported.

# Data handling rules

- Use `pandas` for tabular data when practical.
- Keep column names aligned with the data audit report.
- Do not silently drop columns or rows.
- Do not silently impute missing values unless the method plan or data report allows it.
- Keep feature-target separation explicit for prediction tasks.
- Avoid train-test leakage.
- Keep units and indicator directions consistent with the method plan.

# Plotting rules

- Use `matplotlib` by default.
- Save figures with `plt.savefig(...)`.
- Use readable labels, units, legends, and titles where needed.
- Close figures after saving when generating multiple plots.
- Do not rely on interactive display.

Example:

```python
import matplotlib.pyplot as plt

plt.figure()
plt.plot(x, y)
plt.xlabel("x")
plt.ylabel("y")
plt.tight_layout()
plt.savefig(FIGURE_DIR / "q1_result.png", dpi=300)
plt.close()
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
- Do not use notebook-only workflows.
- Do not use Python-only binary artifacts as the only saved result.
- Do not hide assumptions inside code.
- Do not silently drop rows or columns.
- Do not overwrite raw data.
- Do not claim the code is reviewed or correct before `python-code-reviewer`.

# Verification

Before handoff, verify:

- `implementation.target = python`.
- Every generated script maps to a subquestion and method-plan item.
- Baseline code exists for every main model unless explicitly impossible.
- Scripts are saved under `workspace/code/python/`.
- Cleaned data paths are used.
- No raw data file is overwritten.
- Random seeds are fixed where needed.
- Results are saved under `workspace/results/`.
- Figures are saved under `workspace/figures/`.
- Outputs are portable enough for downstream checks.
- Dependency risks are listed.
- The next skill is `code-reviewer`.

# Failure modes

Stop and report a blocker if:

- The method plan is missing.
- The method plan is not validated.
- `implementation.target` is not `python`.
- Cleaned data is required but unavailable.
- Required data fields are missing.
- The selected method cannot be implemented with reasonable Python dependencies.
- The expected output is not defined.
- Generating code would require inventing data, labels, parameters, or metrics.
- Contest rules disallow Python for official computation and no mixed-language workflow is approved.

# Stop conditions

This skill must stop instead of guessing when:

- Model variables cannot be mapped to cleaned data fields.
- Important parameters are unspecified and cannot be safely defaulted.
- The method requires a heavy dependency that is not approved.
- The code would need to overwrite raw data.
- A script would produce results that cannot be traced to the method plan.
- Continuing would change the mathematical meaning of the model.
- Python is not allowed by contest rules for the intended stage.

When stopping, output:

- blocker
- why it matters
- affected subquestion or model
- missing information or artifact
- safe partial implementation if any
- recommended next skill
- next action

# Handoff

After generating Python scripts, hand off to:

```
code-reviewer
```

The handoff should include:

- generated `.py` script paths
- method plan path
- cleaned data paths
- field mapping
- expected result paths
- expected figure paths
- runtime notes
- dependency notes
- run instructions
- known risks and assumptions

`code-reviewer` should then route the scripts to:

```
python-code-reviewer
```

Do not hand off directly to `robustness-checker`.

# Examples

## Example 1: Generate Python evaluation model code

Input state:

- Q1 uses equal-weight baseline and entropy-weight TOPSIS.
- Cleaned indicator data exists.
- `implementation.target = python`.

Output:

```json
{
  "python_code_generation_summary": {
    "implementation_target": "python",
    "generated_scripts": [
      "workspace/code/python/Q1/q1_baseline.py",
      "workspace/code/python/Q1/q1_entropy_topsis.py"
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
      "python workspace/code/python/Q1/q1_baseline.py",
      "python workspace/code/python/Q1/q1_entropy_topsis.py"
    ],
    "known_risks": [
      "Indicator direction must match the method plan before interpretation."
    ],
    "recommended_next_skill": "code-reviewer"
  }
}
```

## Example 2: Generate Python prediction code

Input state:

- Q2 uses moving-average baseline and random forest regression.
- Cleaned training data exists.
- `implementation.target = python`.

Output:

```json
{
  "python_code_generation_summary": {
    "implementation_target": "python",
    "generated_scripts": [
      "workspace/code/python/Q2/q2_baseline_moving_average.py",
      "workspace/code/python/Q2/q2_random_forest.py"
    ],
    "data_inputs": [
      "workspace/data/data_clean/train_data.csv"
    ],
    "result_outputs": [
      "workspace/results/q2_baseline_metrics.csv",
      "workspace/results/q2_random_forest_metrics.csv",
      "workspace/results/q2_predictions.csv"
    ],
    "figure_outputs": [
      "workspace/figures/q2_prediction_vs_actual.png"
    ],
    "run_instructions": [
      "python workspace/code/python/Q2/q2_baseline_moving_average.py",
      "python workspace/code/python/Q2/q2_random_forest.py"
    ],
    "known_risks": [
      "Train-test split must be checked for leakage during code review."
    ],
    "recommended_next_skill": "code-reviewer"
  }
}
```

## Example 3: Blocked by missing cleaned data

Input state:

- Method plan exists.
- `implementation.target = python`.
- Raw data exists.
- Cleaned data does not exist.

Output:

```json
{
  "blocked_items": [
    "Cleaned data is required before Python code generation."
  ],
  "recommended_next_skill": "data-auditor-cleaner",
  "recommended_next_action": "Audit and clean the raw data, then provide cleaned data paths and field mapping."
}
```

## Example 4: Blocked by unsupported implementation target

Input state:

- Method plan says `implementation.target = matlab`.
- This skill was invoked directly.

Output:

```json
{
  "blocked_items": [
    "python-model-code-generator was invoked, but implementation.target is matlab."
  ],
  "recommended_next_skill": "model-code-analyzer",
  "recommended_next_action": "Route the method plan through model-code-analyzer."
}
```

## Example 5: Blocked by contest language restriction

Input state:

- Contest requires MATLAB / 北太天元 for official computation.
- Method plan says `implementation.target = python`.
- No mixed-language workflow is approved.

Output:

```json
{
  "blocked_items": [
    "Python is not approved for official computation under the contest language requirement."
  ],
  "recommended_next_skill": "method-selector",
  "recommended_next_action": "Change implementation.target to matlab or explicitly define Python as an auxiliary preprocessing branch."
}
```

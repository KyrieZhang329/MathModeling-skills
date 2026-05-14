---
name: python-model-code-generator
description: Generate executable Python modeling code from a validated method plan and cleaned data.
license: MIT
---

# Purpose

Generate Python modeling code from a validated candidate method pool and cleaned data.

This skill implements the approved candidate methods (baseline, main candidates, optional improved) as Python scripts. It should produce code that reads cleaned data, runs the planned models, saves results in the `experiments/roundN/` output structure, generates a `run_summary.json`, and produces portable artifacts for downstream experiment reports, robustness checks, and paper writing.

This skill does not choose the model, clean raw data, review code, run final QA, or write paper sections.

# When to use

Use this skill:

- After `model-code-analyzer` hands off to `python-model-code-generator`.
- When `implementation.target = python`.
- When Python is allowed by the contest or requested by the user.
- When the candidate method pool and cleaned data are ready.
- When executable `.py` scripts are needed for one or more candidate methods in a specific experiment round.

# Preconditions

The following should already exist or be provided:

- A validated problem parse.
- A validated problem classification artifact.
- A candidate method pool at `methods/Qx/qx_method_candidates.md`.
- `implementation.target = python`.
- Cleaned data under `workspace/data/data_clean/`, unless the model does not need tabular data.
- Data audit report and field mapping if data is used.
- `code/model-code-analyzer.md` — the code thinking document specifying round number, methods to implement, and output structure.
- Expected result files and figure files from the candidate method pool.
- Runtime notes if specific dependency or portability constraints exist.

If the candidate method pool is missing, hand back to `method-selector`.

If `code/model-code-analyzer.md` is missing, hand back to `model-code-analyzer`.

If cleaned data is required but missing, hand back to `data-auditor-cleaner`.

If `implementation.target` is not `python`, hand back to `model-code-analyzer`.

# Inputs

Use or request:

- `methods/Qx/qx_method_candidates.md` — candidate method pool for each subquestion.
- `code/model-code-analyzer.md` — code thinking document.
- `workspace/data/data_report.md`, if available.
- Cleaned data under `workspace/data/data_clean/`.
- Field mapping from the data audit stage.
- Required model inputs per candidate method.
- Expected result outputs per candidate method.
- Expected figure outputs per candidate method.
- The round number (N for `roundN`).
- Runtime notes such as:
  - `minimal-dependencies`
  - `portable-artifacts`
  - `avoid-notebook-only-code`
  - `save-csv-json-png`
  - `fixed-random-seed`

# Workflow

1. Read the code thinking document and candidate method pool.
   - Identify the target subquestion(s) and round number.
   - Identify which methods are to be implemented in this round (baseline, main candidates, optional improved).
   - Identify required inputs, expected outputs, validation needs, and robustness needs per method.
   - Do not change the selected methods or modeling route.

2. Confirm Python target.
   - Confirm `implementation.target = python`.
   - Preserve all runtime notes.
   - Use a minimal, common scientific Python stack (`numpy`, `pandas`, `scipy`, `scikit-learn`, `matplotlib`).

3. Check data readiness.
   - Use cleaned data from `workspace/data/data_clean/`.
   - Do not read from `workspace/data/data_raw/` unless only inspecting metadata is explicitly requested.
   - Do not overwrite raw data.
   - Confirm required fields match the data audit report.

4. Plan script structure per subquestion.
   - Save Python code under `code/Qx/` (e.g., `code/Q1/`, `code/Q2/`).
   - One script per candidate method: `qx_m1_baseline.py`, `qx_m2_main.py`, `qx_m3_improved.py`.
   - A `README.md` in each code folder: run order, inputs, outputs, dependencies.
   - A `run_all.py` only if batch execution is useful.
   - Prefer plain `.py` scripts over notebook-only workflows.
   - Keep scripts runnable from the project root.
   - Use small helper functions when they improve clarity.

5. Generate code for each candidate method.
   - **Input**: Read from `workspace/data/data_clean/`.
   - **Processing**: Implement the mathematical logic from the candidate method pool.
   - **Save results**: Save to `results/Qx/experiments/roundN/tables/`.
   - **Save figures**: Save to `results/Qx/experiments/roundN/figures/`.
   - **Save metrics**: Save to `results/Qx/experiments/roundN/metrics/`.
   - **Write log**: Write execution log to `results/Qx/experiments/roundN/logs/`.
   - **Write run_summary.json**: At end of `run_all.py` or the last script, generate `results/Qx/experiments/roundN/run_summary.json`.

6. Implement baseline code first.
   - Implement the baseline method first.
   - Save baseline outputs separately (e.g., `m1_scores.csv`).
   - Make baseline outputs comparable with other methods.

7. Implement main and optional methods.
   - Implement each candidate method with the same output conventions.
   - Use variable names that match the candidate method specification where practical.
   - Save intermediate values needed for explanation or robustness checks.

8. Implement run_summary.json generation.
   - After all methods have executed, write `run_summary.json` with:
     ```python
     import json
     from datetime import datetime

     run_summary = {
         "question": "Q1",
         "round": "round1",
         "execution_timestamp": datetime.now().isoformat(),
         "implementation_target": "python",
         "random_seed": SEED,
         "methods": [
             {
                 "method_id": "M1",
                 "method_name": "equal_weight_scoring",
                 "script": "code/Q1/q1_m1_baseline.py",
                 "status": "success",
                 "execution_time_seconds": 12.5,
                 "input_files": ["workspace/data/data_clean/indicator_data.csv"],
                 "output_files": ["results/Q1/experiments/round1/tables/m1_scores.csv"],
                 "figure_files": ["results/Q1/experiments/round1/figures/m1_score_bar.png"],
                 "metrics_summary": {"score_range": [0.45, 0.92], "ranking_top3": ["A", "B", "C"]},
                 "errors": [],
                 "warnings": []
             }
         ],
         "data_inputs": ["workspace/data/data_clean/indicator_data.csv"],
         "environment": {
             "python_version": "3.10",
             "key_dependencies": ["numpy", "pandas", "matplotlib", "scipy"]
         }
     }
     with open('results/Q1/experiments/round1/run_summary.json', 'w') as f:
         json.dump(run_summary, f, indent=2, ensure_ascii=False)
     ```

9. Add a README.md per code folder.
   - State the subquestion and round.
   - List scripts and their roles.
   - State run order.
   - List expected input data files.
   - List expected output paths.
   - State required dependencies.
   - State known portability risks.

10. Hand off to `code-reviewer`.
    - The router should route `.py` scripts to `python-code-reviewer`.

# Outputs

Produce Python implementation artifacts such as:

- `code/Q1/README.md`
- `code/Q1/q1_m1_baseline.py`
- `code/Q1/q1_m2_main.py`
- `code/Q1/q1_m3_improved.py` (if applicable)
- `code/Q1/run_all.py` (if useful)
- `results/Q1/experiments/roundN/` (created by running the scripts):
  - `tables/*.csv`
  - `figures/*.png`
  - `metrics/*.json`
  - `logs/*.log`
  - `run_summary.json`

# Output format

Prefer this JSON-compatible summary:

```json
{
  "python_code_generation_summary": {
    "implementation_target": "python",
    "subquestion": "Q1",
    "round": "round1",
    "runtime_notes": ["minimal-dependencies", "portable-artifacts"],
    "generated_scripts": [
      "code/Q1/q1_m1_baseline.py",
      "code/Q1/q1_m2_main.py",
      "code/Q1/run_all.py"
    ],
    "data_inputs": [
      "workspace/data/data_clean/indicator_data.csv"
    ],
    "result_outputs": [
      "results/Q1/experiments/round1/tables/m1_scores.csv",
      "results/Q1/experiments/round1/tables/m2_scores.csv",
      "results/Q1/experiments/round1/metrics/m1_metrics.json",
      "results/Q1/experiments/round1/metrics/m2_metrics.json"
    ],
    "figure_outputs": [
      "results/Q1/experiments/round1/figures/m1_score_bar.png",
      "results/Q1/experiments/round1/figures/m2_weight_bar.png"
    ],
    "run_instructions": [
      "cd code/Q1 && python run_all.py"
    ],
    "dependencies": ["numpy", "pandas", "matplotlib", "scipy"],
    "known_risks": [
      "Column names must match the data audit report before interpretation."
    ],
    "recommended_next_skill": "code-reviewer"
  }
}
```

# Python coding rules

Use clear, minimal, reviewable Python code.

Prefer:
- plain `.py` scripts
- `pathlib.Path`
- `numpy`, `pandas`, `scipy`, `scikit-learn` (when justified), `matplotlib`
- relative paths from project root
- explicit result saving to `results/Qx/experiments/roundN/`
- fixed random seeds (`SEED = 2026`)
- portable outputs: `.csv`, `.json`, `.png`
- `run_summary.json` generation

Avoid unless explicitly approved:
- notebook-only code
- hard-coded local absolute paths
- hidden data cleaning inside modeling scripts
- heavy deep learning frameworks (`torch`, `tensorflow`)
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

Avoid heavy dependencies unless explicitly justified:
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
code/Qx/
results/Qx/experiments/roundN/
  ├── tables/
  ├── figures/
  ├── metrics/
  ├── logs/
  └── run_summary.json
```

Avoid hard-coded absolute paths.

Example path setup:
```python
from pathlib import Path

ROOT = Path(".")
DATA_DIR = ROOT / "workspace" / "data" / "data_clean"
RESULT_DIR = ROOT / "results" / "Q1" / "experiments" / "round1"
TABLE_DIR = RESULT_DIR / "tables"
FIGURE_DIR = RESULT_DIR / "figures"
METRIC_DIR = RESULT_DIR / "metrics"
LOG_DIR = RESULT_DIR / "logs"

for d in [TABLE_DIR, FIGURE_DIR, METRIC_DIR, LOG_DIR]:
    d.mkdir(parents=True, exist_ok=True)
```

# Artifact rules

- Read cleaned data from `workspace/data/data_clean/`.
- Do not modify files under `workspace/data/data_raw/`.
- Save Python scripts under `code/Qx/`.
- Save numeric outputs under `results/Qx/experiments/roundN/tables/`.
- Save figures under `results/Qx/experiments/roundN/figures/`.
- Save metrics under `results/Qx/experiments/roundN/metrics/`.
- Save logs under `results/Qx/experiments/roundN/logs/`.
- Generate `run_summary.json` under `results/Qx/experiments/roundN/`.
- Use clear file names with method prefix:
  - `m1_scores.csv`, `m2_weights.csv`, `m1_metrics.json`
  - `m1_score_bar.png`, `m2_weight_distribution.png`
- Separate outputs from different methods clearly.
- Save important intermediate results if later paper explanation or robustness checks require them.

# Randomness rules

If randomness is used:
- Set fixed seeds: `random.seed(SEED)`, `np.random.seed(SEED)`.
- Record the seed in comments and `run_summary.json`.
- For scikit-learn: pass `random_state=SEED` where supported.

```python
import random
import numpy as np

SEED = 2026
random.seed(SEED)
np.random.seed(SEED)
```

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
- Save figures with `plt.savefig(FIGURE_DIR / "m1_result.png", dpi=300, bbox_inches='tight')`.
- Use `svg.fonttype = 'none'` for editable SVG text.
- Use readable labels, units, legends, and titles.
- Close figures after saving: `plt.close(fig)`.
- Do not rely on interactive display.

```python
import matplotlib.pyplot as plt

plt.rcParams['svg.fonttype'] = 'none'
fig, ax = plt.subplots()
ax.plot(x, y)
ax.set_xlabel("x")
ax.set_ylabel("y")
fig.tight_layout()
fig.savefig(FIGURE_DIR / "m1_result.png", dpi=300, bbox_inches='tight')
fig.savefig(FIGURE_DIR / "m1_result.svg", bbox_inches='tight')
plt.close(fig)
```

# Commenting rules

Use comments to explain:
- data inputs and their source paths
- key variables and their meanings
- model steps and non-obvious formulas
- output files and their destinations
- assumptions inherited from the method plan
- random seed and reproducibility notes

Avoid over-commenting trivial syntax.

# Rules

- Do not select or change the model — implement what the candidate method pool specifies.
- Do not clean raw data.
- Do not fabricate data, labels, parameters, metrics, or results.
- Do not write paper text.
- Do not perform final QA.
- Do not use notebook-only workflows.
- Do not use Python-only binary artifacts as the only saved result.
- Do not hide assumptions inside code.
- Do not silently drop rows or columns.
- Do not overwrite raw data.
- Always generate `run_summary.json`.
- Always create the full `experiments/roundN/` directory structure.
- Do not claim the code is reviewed or correct before `python-code-reviewer`.
- Separate baseline outputs from main method outputs.

# Verification

Before handing off, verify:

- `implementation.target = python`.
- Every generated script maps to a method in the candidate method pool.
- Baseline code exists for every subquestion unless explicitly impossible.
- Scripts are saved under `code/Qx/`.
- Scripts write outputs to `results/Qx/experiments/roundN/`.
- Cleaned data paths are used.
- No raw data file is overwritten.
- Random seeds are fixed where needed.
- `run_summary.json` generation is implemented.
- Figures are saved to `results/Qx/experiments/roundN/figures/`.
- Outputs are portable enough for downstream checks.
- A `README.md` exists in the code folder.
- Dependency risks are listed.
- The next skill is `code-reviewer`.

# Failure modes

Stop and report a blocker if:

- The candidate method pool is missing.
- `code/model-code-analyzer.md` is missing.
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
- A script would produce results that cannot be traced to the candidate method pool.
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

`code-reviewer`

The handoff should include:
- generated `.py` script paths
- code folder `README.md` path
- candidate method pool path
- cleaned data paths
- field mapping
- expected result paths under `results/Qx/experiments/roundN/`
- expected figure paths
- runtime notes
- dependency notes
- run instructions
- known risks and assumptions

`code-reviewer` should route the scripts to `python-code-reviewer`.

Do not hand off directly to `robustness-checker` or `result-report-generator`.

# Examples

## Example 1: Generate Python evaluation model code for round 1

Input state:
- Q1 candidate pool: M1 (equal-weight baseline), M2 (entropy-TOPSIS).
- Cleaned indicator data exists.
- Round: round1.
- `implementation.target = python`.

Output:
```json
{
  "python_code_generation_summary": {
    "implementation_target": "python",
    "subquestion": "Q1",
    "round": "round1",
    "generated_scripts": [
      "code/Q1/q1_m1_baseline.py",
      "code/Q1/q1_m2_entropy_topsis.py",
      "code/Q1/run_all.py",
      "code/Q1/README.md"
    ],
    "data_inputs": [
      "workspace/data/data_clean/indicator_data.csv"
    ],
    "result_outputs": [
      "results/Q1/experiments/round1/tables/m1_equal_weight_scores.csv",
      "results/Q1/experiments/round1/tables/m2_entropy_topsis_scores.csv",
      "results/Q1/experiments/round1/tables/m2_weights.csv",
      "results/Q1/experiments/round1/metrics/m1_metrics.json",
      "results/Q1/experiments/round1/metrics/m2_metrics.json",
      "results/Q1/experiments/round1/run_summary.json"
    ],
    "figure_outputs": [
      "results/Q1/experiments/round1/figures/m1_score_bar.png",
      "results/Q1/experiments/round1/figures/m2_weight_bar.png",
      "results/Q1/experiments/round1/figures/m2_score_dist.png"
    ],
    "run_instructions": [
      "cd code/Q1 && python run_all.py"
    ],
    "known_risks": [
      "Indicator direction must match the method plan before interpretation."
    ],
    "recommended_next_skill": "code-reviewer"
  }
}
```

## Example 2: Blocked by missing candidate pool

```json
{
  "blocked_items": [
    "The candidate method pool is missing at methods/Q1/q1_method_candidates.md. Cannot determine which methods to implement."
  ],
  "recommended_next_skill": "method-selector",
  "recommended_next_action": "Generate the candidate method pool for Q1 before code generation."
}
```

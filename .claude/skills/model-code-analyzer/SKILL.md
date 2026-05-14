---
name: model-code-analyzer
description: Translate a validated method plan into language-neutral code logic, folder layout, and handoff notes before Python or MATLAB code generation.
license: MIT
---

# Purpose

Translate validated mathematical modeling plans into executable code thinking.

This skill does not write runnable code itself. It turns the selected modeling route (from the candidate method pool) into a language-neutral coding plan: what each subquestion needs, which data enters each stage, which intermediate artifacts should be saved, how code should be split by subquestion, how the experiments/roundN/ output structure must be organized, and which language-specific generator should implement the plan next.

A critical function of this skill is defining the output structure so that experiment reproducibility, multi-method comparison, and downstream report generation are possible.

# When to use

Use this skill:

- After `data-auditor-cleaner` has confirmed that required data is ready or ready with documented warnings.
- After `method-selector` has produced the candidate method pool per subquestion.
- Before `python-model-code-generator` or `matlab-model-code-generator`.
- When the team needs a code-thinking document under `code/` before writing executable scripts.
- When a new round of experiments is being planned and the code structure needs to be defined.

# Preconditions

The following should already exist or be provided:

- Candidate method pool per subquestion (`methods/Qx/qx_method_candidates.md`).
- An `implementation.target` field with value `python` or `matlab`.
- Cleaned data under `workspace/data/data_clean/`, if data is required.
- A data report at `workspace/data/data_report.md`, if data is used.
- The round number for this experiment batch (round1, round2, ..., final).

# Inputs

Use or request:

- `methods/Qx/qx_method_candidates.md` — candidate method pool.
- `workspace/data/data_report.md`, if available.
- Cleaned data under `workspace/data/data_clean/`.
- Implementation target and runtime notes.
- Expected result outputs and figure outputs.
- Round number (N for roundN).

# Workflow

1. Read the candidate method pool.
   - Work subquestion by subquestion.
   - Identify which methods are prioritized for this round.
   - Preserve baseline and candidate method roles.
   - Do not change the method list.

2. Translate the mathematical plan into code logic per subquestion.
   - Identify required inputs for each candidate method.
   - Identify calculation order, intermediate values, and saved artifacts.
   - Identify which outputs belong under `results/Qx/experiments/roundN/`.
   - For each method, plan the computation pipeline.

3. Define the experiments/roundN/ output structure.
   - Every round of experiments MUST produce outputs in this structure:
     ```
     results/Qx/experiments/roundN/
     ├── figures/          # All generated figures (.png, .pdf)
     │   ├── m1_result.png
     │   ├── m2_result.png
     │   └── comparison.png
     ├── tables/           # All result tables (.csv)
     │   ├── m1_scores.csv
     │   ├── m2_scores.csv
     │   └── comparison_table.csv
     ├── metrics/          # All evaluation metrics (.csv, .json)
     │   ├── m1_metrics.json
     │   ├── m2_metrics.json
     │   └── comparison_metrics.json
     ├── logs/             # Execution logs (.log, .txt)
     │   ├── m1_run.log
     │   └── m2_run.log
     └── run_summary.json  # Execution record (see specification below)
     ```
   - Ensure this structure is documented and expected by downstream skills.

4. Specify the run_summary.json.
   - This is the machine-readable record of what was executed.
   - Downstream skills (`result-report-generator`, `robustness-checker`) rely on it.
   - Required fields:
     ```json
     {
       "question": "Q1",
       "round": "round1",
       "execution_timestamp": "2026-05-14T10:30:00+08:00",
       "implementation_target": "python",
       "random_seed": 2026,
       "methods": [
         {
           "method_id": "M1",
           "method_name": "equal_weight_scoring",
           "script": "code/Q1/q1_baseline.py",
           "status": "success",
           "execution_time_seconds": 12.5,
           "input_files": [
             "workspace/data/data_clean/indicator_data.csv"
           ],
           "output_files": [
             "results/Q1/experiments/round1/tables/m1_scores.csv",
             "results/Q1/experiments/round1/metrics/m1_metrics.json"
           ],
           "figure_files": [
             "results/Q1/experiments/round1/figures/m1_score_bar.png"
           ],
           "metrics_summary": {
             "score_range": [0.45, 0.92],
             "ranking_top3": ["City_A", "City_B", "City_C"]
           },
           "errors": [],
           "warnings": []
         },
         {
           "method_id": "M2",
           "method_name": "entropy_topsis",
           "script": "code/Q1/q1_entropy_topsis.py",
           "status": "success",
           "execution_time_seconds": 18.3,
           "input_files": [
             "workspace/data/data_clean/indicator_data.csv"
           ],
           "output_files": [
             "results/Q1/experiments/round1/tables/m2_scores.csv",
             "results/Q1/experiments/round1/tables/m2_weights.csv",
             "results/Q1/experiments/round1/metrics/m2_metrics.json"
           ],
           "figure_files": [
             "results/Q1/experiments/round1/figures/m2_weight_bar.png",
             "results/Q1/experiments/round1/figures/m2_score_dist.png"
           ],
           "metrics_summary": {
             "entropy_weights": {"indicator_1": 0.23, "indicator_2": 0.18, "indicator_3": 0.31, "indicator_4": 0.28},
             "closeness_scores_range": [0.12, 0.88],
             "ranking_top3": ["City_A", "City_C", "City_B"]
           },
           "errors": [],
           "warnings": ["Indicator_4 has very high variance and dominates entropy weight."]
         }
       ],
       "comparison": {
         "ranking_correlation": 0.87,
         "top3_overlap": "2 out of 3 identical"
       },
       "data_inputs": [
         "workspace/data/data_clean/indicator_data.csv"
       ],
       "environment": {
         "python_version": "3.10",
         "key_dependencies": ["numpy==1.24", "pandas==1.5", "matplotlib==3.6"]
       }
     }
     ```

5. Define the code layout per subquestion.
   - For Python: `code/Q1/`, `code/Q2/`, etc.
   - For MATLAB: `code/matlab/Q1/`, `code/matlab/Q2/`, etc.
   - One script per candidate method within each subquestion folder.
   - A `README.md` in each code subfolder documenting run order, inputs, outputs, and dependencies.
   - A `run_all.py` or `run_all.m` script only if batch execution is useful.

6. Write the code-thinking document.
   - Save the analysis under `code/model-code-analyzer.md`.
   - Keep the document language-neutral.
   - Make the handoff usable by the language-specific code generator.

7. Hand off to the language-specific generator.
   - Route to `python-model-code-generator` when `implementation.target = python`.
   - Route to `matlab-model-code-generator` when `implementation.target = matlab`.

# Outputs

Produce one Markdown planning artifact:

- `code/model-code-analyzer.md`

The document should include:

- Implementation target and runtime notes.
- Round number and purpose.
- Per-subquestion code logic for each candidate method.
- Expected code folder layout.
- Expected experiments/roundN/ output structure (explicit directory tree).
- run_summary.json field specification.
- Expected input data per method.
- Expected result artifacts per method.
- Expected figure artifacts per method.
- Reproducibility requirements (fixed seeds, environment notes).
- Recommended next skill.

# Output format

Write a concise Markdown document under `code/model-code-analyzer.md`.

The document should explain, for each subquestion:

```markdown
# Model Code Analysis

## Implementation Target
- **Language**: python / matlab
- **Round**: roundN (purpose: e.g., "first multi-method comparison")
- **Runtime notes**: [constraints like minimal-dependencies, portable-artifacts, etc.]

## Qx: [Subquestion Summary]

### Methods to Implement This Round
- M1: [name] (baseline / main candidate)
- M2: [name] (main candidate / backup)
- M3: [name] (if applicable)

### Per-Method Code Logic

#### M1: [name]
1. **Input**: read `data_clean/[file]`, columns [list]
2. **Processing**: [step-by-step computation logic]
3. **Output**: save results to `results/Qx/experiments/roundN/tables/m1_results.csv`
4. **Figures**: save to `results/Qx/experiments/roundN/figures/m1_*.png`
5. **Metrics**: save to `results/Qx/experiments/roundN/metrics/m1_metrics.json`
6. **Log**: write execution log to `results/Qx/experiments/roundN/logs/m1_run.log`

#### M2: [name]
[same structure]

### Expected Code Files
```
code/Qx/
├── README.md           # Run instructions
├── qx_baseline.py      # M1 implementation
├── qx_main.py          # M2 implementation
└── qx_improved.py      # M3 implementation (if applicable)
```

### Expected Output Structure
```
results/Qx/experiments/roundN/
├── figures/
│   ├── m1_score_bar.png
│   ├── m2_weight_bar.png
│   ├── m2_score_dist.png
│   └── ranking_comparison.png
├── tables/
│   ├── m1_scores.csv
│   ├── m2_scores.csv
│   ├── m2_weights.csv
│   └── comparison_table.csv
├── metrics/
│   ├── m1_metrics.json
│   ├── m2_metrics.json
│   └── comparison_metrics.json
├── logs/
│   ├── m1_run.log
│   └── m2_run.log
└── run_summary.json
```

### run_summary.json Key Fields
[Reference the specification above]

### Reproducibility
- **Random seed**: 2026 (fixed)
- **Environment**: Python 3.10, numpy, pandas, matplotlib, scipy
- **Expected runtime**: ~2 minutes for all methods
```

# Rules

- Do not write executable Python or MATLAB code in this skill.
- Do not change the candidate method pool or method specifications.
- Do not fabricate data, scripts, results, or figures.
- Do not bypass language-specific code generators.
- Always specify the full experiments/roundN/ output structure.
- Always specify the run_summary.json schema.
- Keep the code plan minimal but complete enough for the code generator to execute.
- Ensure every method has clearly defined inputs, processing steps, and output destinations.
- Separate baseline outputs from main method outputs in the plan.
- Include reproducibility requirements: random seed, environment, expected runtime.

# Verification

Before handing off, verify:

- Every method from the candidate pool for this round has a code logic plan.
- The experiments/roundN/ output structure is explicitly specified.
- The run_summary.json schema is referenced or specified.
- Input data paths are correct.
- Output paths follow the `results/Qx/experiments/roundN/` convention.
- Code folder layout follows `code/Qx/` (Python) or `code/matlab/Qx/` (MATLAB) convention.
- Reproducibility requirements are stated.
- The next skill is the correct language-specific generator.

# Failure modes

Stop and report a blocker if:

- The candidate method pool is missing.
- `implementation.target` is undefined.
- Cleaned data is required but unavailable.
- The methods cannot be mapped to available data fields.
- The expected outputs are undefined.

# Stop conditions

This skill must stop instead of guessing when:

- Important parameters are unspecified and cannot be safely defaulted.
- The method requires a dependency that may be unavailable.
- The code would need to overwrite raw data.
- A script would produce results that cannot be traced to the candidate method.
- Continuing would change the mathematical meaning of the model.

When stopping, output:

- blocker
- why it matters
- affected subquestion or model
- missing information or artifact
- safe partial plan if any
- recommended next skill

# Handoff

After writing `code/model-code-analyzer.md`, hand off to:

- `python-model-code-generator` if `implementation.target = python`
- `matlab-model-code-generator` if `implementation.target = matlab`

The handoff must include:

- `code/model-code-analyzer.md` (the full code thinking document)
- Candidate method pool files
- Cleaned data paths
- Round number
- experiments/roundN/ output structure specification
- run_summary.json schema
- Reproducibility requirements

# Examples

## Example 1: Python evaluation model code analysis for round 1

Input state:
- Q1 candidate pool: M1 (equal-weight baseline), M2 (entropy-TOPSIS).
- Implementation target: python.
- Round: round1 (first multi-method comparison).
- Data: cleaned indicator matrix.

Output (`code/model-code-analyzer.md` excerpt):
```markdown
## Q1: City Resilience Evaluation

### Methods to Implement This Round
- M1: equal_weight_scoring (baseline)
- M2: entropy_topsis (main candidate)

### Expected Output Structure
```
results/Q1/experiments/round1/
├── figures/
│   ├── m1_score_bar.png
│   ├── m2_weight_bar.png
│   ├── m2_score_dist.png
│   └── ranking_comparison.png
├── tables/
│   ├── m1_scores.csv
│   ├── m2_scores.csv
│   └── m2_weights.csv
├── metrics/
│   ├── m1_metrics.json
│   ├── m2_metrics.json
│   └── comparison_metrics.json
├── logs/
│   ├── m1_run.log
│   └── m2_run.log
└── run_summary.json
```

### run_summary.json
[Full specification with M1 and M2 method entries, comparison field, environment info]
```

## Example 2: MATLAB optimization model code analysis

Input state:
- Q3 candidate pool: M1 (greedy baseline), M2 (integer linear programming).
- Implementation target: matlab.
- Runtime notes: beita-tianyuan-compatible.

Output: same structure as above, but code under `code/matlab/Q3/`, and MATLAB-specific output path conventions.

# Implementation Targets

This document defines how `MathModeling-skills` handles code implementation targets.

The repository keeps the modeling workflow unified until the code stage. It branches only when code must be generated or reviewed, then merges back into the shared result, robustness, figure, paper, and QA workflow.

## Supported targets

The repository supports two implementation targets:

```text
python
matlab
```

`matlab` includes MATLAB-compatible code and MATLAB / 北太天元 compatible code.

北太天元 should not be treated as a separate third target. Use:

```json
{
  "implementation": {
    "target": "matlab",
    "runtime_notes": [
      "beita-tianyuan-compatible"
    ]
  }
}
```

## Why only two targets

The modeling logic does not depend on the programming language.

These stages should remain language-neutral:

```text
problem-parser
problem-classifier
method-selector
data-auditor-cleaner
robustness-checker
figure-table-planner
paper-section-writer
quality-assurance-auditor
```

The language matters mainly in two places:

```text
model-code-generator
code-reviewer
```

Code generation needs language-specific syntax, dependencies, file formats, and runtime assumptions.

Code review needs language-specific checks, such as Python package imports or MATLAB row/column vector consistency.

After code review, downstream skills should consume saved artifacts rather than caring whether they were produced by Python or MATLAB.

## Workflow shape

The intended workflow is:

```text
data-auditor-cleaner
→ model-code-generator
   ├── python-model-code-generator
   └── matlab-model-code-generator
→ code-reviewer
   ├── python-code-reviewer
   └── matlab-code-reviewer
→ robustness-checker
→ figure-table-planner
→ paper-section-writer
→ quality-assurance-auditor
```

`model-code-generator` and `code-reviewer` are router skills.

They should decide where to send the task. They should not contain detailed language-specific implementation logic.

## The `implementation` field

A validated method plan should include an `implementation` object.

Recommended structure:

```json
{
  "implementation": {
    "target": "matlab",
    "runtime_notes": [
      "beita-tianyuan-compatible",
      "avoid-heavy-toolboxes",
      "prefer-basic-matrix-and-table-operations"
    ],
    "script_style": "script",
    "result_format": [
      "csv",
      "mat"
    ],
    "figure_format": [
      "png"
    ],
    "random_seed_required": true
  }
}
```

### `target`

Allowed values:

```text
python
matlab
```

Use `python` when Python is the approved implementation language.

Use `matlab` when the contest requires MATLAB, 北太天元, or MATLAB-compatible scripts.

### `runtime_notes`

Use `runtime_notes` to record runtime constraints.

Common Python notes:

```text
minimal-dependencies
portable-artifacts
avoid-notebook-only-code
save-csv-json-png
fixed-random-seed
```

Common MATLAB / 北太天元 notes:

```text
beita-tianyuan-compatible
avoid-heavy-toolboxes
prefer-basic-matrix-and-table-operations
avoid-live-scripts
avoid-app-designer
save-portable-artifacts
```

### `script_style`

Suggested values:

```text
script
function
mixed
```

For mathematical modeling contests, `script` is usually the safest default.

Use `function` only when the function structure makes the code easier to run and review.

### `result_format`

Recommended portable result formats:

```text
csv
json
mat
xlsx
```

For MATLAB / 北太天元 workflows, prefer:

```text
csv
mat
```

For Python workflows, prefer:

```text
csv
json
```

### `figure_format`

Recommended figure formats:

```text
png
pdf
fig
```

Use `png` as the default because it is portable and easy to include in papers.

### `random_seed_required`

Use `true` when the method contains randomness, including:

- Monte Carlo simulation
- random initialization
- random train-test split
- stochastic optimization
- machine learning models with random states

## Routing rules

### Route to Python

Route to `python-model-code-generator` when:

```json
{
  "implementation": {
    "target": "python"
  }
}
```

Expected script directory:

```text
workspace/scripts/python/
```

Expected output directories:

```text
workspace/results/
workspace/figures/
```

### Route to MATLAB / 北太天元

Route to `matlab-model-code-generator` when:

```json
{
  "implementation": {
    "target": "matlab"
  }
}
```

Expected script directory:

```text
workspace/scripts/matlab/
```

Expected output directories:

```text
workspace/results/
workspace/figures/
```

## Workspace convention

Recommended workspace layout:

```text
workspace/
├── problem/
├── data_raw/
├── data_clean/
├── scripts/
│   ├── python/
│   └── matlab/
├── results/
├── figures/
├── paper_sections/
└── final_paper/
```

Language-specific code should stay under:

```text
workspace/scripts/python/
workspace/scripts/matlab/
```

Results and figures should remain language-neutral:

```text
workspace/results/
workspace/figures/
```

This allows downstream skills to work from artifacts instead of language-specific code.

## Artifact naming

Use clear names that include:

- subquestion id
- model role
- artifact type

Examples:

```text
workspace/scripts/matlab/q1_baseline.m
workspace/scripts/matlab/q1_main.m
workspace/scripts/python/q2_baseline.py
workspace/scripts/python/q2_main.py

workspace/results/q1_baseline_results.csv
workspace/results/q1_main_results.csv
workspace/results/q2_prediction_metrics.csv
workspace/results/q2_predictions.csv

workspace/figures/q1_ranking_comparison.png
workspace/figures/q2_prediction_vs_actual.png
```

Avoid vague names:

```text
result.csv
new_result.csv
final.csv
test.py
main2.m
```

## Mixed-language workflows

A mixed-language workflow is allowed only when it is explicit in the method plan.

Example:

```json
{
  "implementation": {
    "target": "matlab",
    "runtime_notes": [
      "beita-tianyuan-compatible",
      "python-used-for-preprocessing-only"
    ]
  }
}
```

In this case:

- Python preprocessing scripts should be saved under `workspace/scripts/python/`.
- MATLAB official modeling scripts should be saved under `workspace/scripts/matlab/`.
- The handoff from Python to MATLAB must be through portable data files under `workspace/data_clean/` or `workspace/results/`.
- The final official computation source must be clear.

Do not create a mixed-language workflow accidentally.

If the contest requires MATLAB / 北太天元 for official computation, Python should not become the hidden source of final results unless explicitly approved.

## Downstream merge

After language-specific code review, the workflow merges back into:

```text
robustness-checker
```

At that point, downstream skills should use:

```text
workspace/results/
workspace/figures/
```

They should not depend on whether the result came from Python or MATLAB.

Downstream claims should be based on saved result artifacts, not on console output.

## Common failure cases

### Missing implementation target

Problem:

```json
{
  "implementation": {}
}
```

Action:

Return to `method-selector` and add:

```json
{
  "implementation": {
    "target": "python"
  }
}
```

or:

```json
{
  "implementation": {
    "target": "matlab"
  }
}
```

### Contest requires MATLAB / 北太天元 but target is Python

Problem:

```json
{
  "implementation": {
    "target": "python"
  }
}
```

when contest rules require MATLAB / 北太天元.

Action:

Return to `method-selector` and revise the target to:

```json
{
  "implementation": {
    "target": "matlab",
    "runtime_notes": [
      "beita-tianyuan-compatible"
    ]
  }
}
```

### Script language conflicts with target

Problem:

```text
implementation.target = matlab
only .py scripts exist
```

Action:

Return to `model-code-generator` and regenerate MATLAB scripts, or return to `method-selector` if Python was intended.

### Results are saved only in language-specific binary formats

Problem:

```text
only .pkl or only environment-specific files are saved
```

Action:

Save portable artifacts such as:

```text
csv
json
mat
png
```

## Practical rule

Select the implementation target in the method plan.

Keep code language-specific.

Keep results language-neutral.






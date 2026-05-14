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

These stages remain language-neutral:

```text
problem-parser
problem-classifier
related-paper-analyzer
method-selector
symbol-table-builder
model-assumptions-builder
data-auditor-cleaner
model-code-analyzer
result-report-generator
robustness-checker
final-method-explainer
figure-table-planner
solution-package-builder
paper-section-writer
paper-polisher
reference-manager
quality-assurance-auditor
workflow-orchestrator
```

The language matters mainly in two places:

```text
python-model-code-generator (or matlab-model-code-generator)
code-reviewer → python-code-reviewer (or matlab-code-reviewer)
```

Code generation needs language-specific syntax, dependencies, file formats, and runtime assumptions.

Code review needs language-specific checks, such as Python package imports or MATLAB row/column vector consistency.

After code review, downstream skills should consume saved artifacts rather than caring whether they were produced by Python or MATLAB.

## Workflow shape

The intended code-related workflow is:

```text
model-code-analyzer
→ python-model-code-generator (when target = python)
   or matlab-model-code-generator (when target = matlab)
→ code-reviewer (router)
   ├── python-code-reviewer
   └── matlab-code-reviewer
→ result-report-generator
→ robustness-checker
```

`model-code-analyzer` produces the language-neutral code thinking document.
`python-model-code-generator` and `matlab-model-code-generator` implement it.
`code-reviewer` routes to the correct language-specific reviewer.

## The `implementation` field

A validated candidate method pool should include an `implementation` object.

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
    "result_format": ["csv", "mat"],
    "figure_format": ["png", "svg"],
    "random_seed_required": true
  }
}
```

### `target`

Allowed values: `python`, `matlab`.

### `runtime_notes`

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

## Code and output paths

Python code goes under:
```text
code/Qx/
```

MATLAB code goes under:
```text
code/matlab/Qx/
```

All code must output to the language-neutral structure:
```text
results/Qx/experiments/roundN/
├── tables/
├── figures/
├── metrics/
├── logs/
└── run_summary.json
```

This allows downstream skills to work from artifacts instead of language-specific code.

## Artifact naming

Use clear names that include method prefix:

```text
# Python
code/Q1/q1_m1_baseline.py
code/Q1/q1_m2_main.py

# MATLAB
code/matlab/Q1/q1_m1_baseline.m
code/matlab/Q1/q1_m2_main.m

# Outputs
results/Q1/experiments/round1/tables/m1_scores.csv
results/Q1/experiments/round1/tables/m2_scores.csv
results/Q1/experiments/round1/figures/m1_score_bar.png
results/Q1/experiments/round1/run_summary.json
```

Avoid vague names:
```text
result.csv
final.csv
test.py
main2.m
```

## Mixed-language workflows

A mixed-language workflow is allowed only when it is explicit in the method plan.

In this case:
- Python preprocessing scripts → `code/Qx/`
- MATLAB official modeling scripts → `code/matlab/Qx/`
- Handoff must be through portable data files under `workspace/data_clean/`
- The final official computation source must be clear

Do not create a mixed-language workflow accidentally.

## Downstream merge

After code review, the workflow merges back into language-neutral skills that consume:
```text
results/Qx/experiments/roundN/
robustness/Qx/
```

Downstream claims should be based on saved result artifacts, not on console output.

## Practical rule

Select the implementation target in the candidate method pool.
Keep code language-specific.
Keep results language-neutral.
Always generate `run_summary.json`.

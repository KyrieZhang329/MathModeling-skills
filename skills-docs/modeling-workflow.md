# Modeling Workflow

[English](./modeling-workflow.md) | [简体中文](./modeling-workflow-zh.md)

This document explains how the 24 skills in `MathModeling-skills` are connected during a mathematical modeling contest.

The goal is not to force every contest problem into the same solution pattern. The goal is to reduce common workflow mistakes: reading the problem too loosely, choosing methods too early, writing code before the model is clear, writing paper conclusions before results exist, or writing claims that cannot be traced back to result files.

## Workflow overview

```text
workflow-orchestrator
→ problem-parser
→ problem-classifier
→ related-paper-analyzer
→ method-selector
→ symbol-table-builder
→ model-assumptions-builder
→ data-auditor-cleaner
→ model-code-analyzer
   ├── python-model-code-generator
   └── matlab-model-code-generator
→ code-reviewer
   ├── python-code-reviewer
   └── matlab-code-reviewer
→ result-report-generator
→ robustness-checker
→ final-method-explainer
→ figure-table-planner
→ math-figure-generator
→ solution-package-builder
→ paper-section-writer
→ paper-polisher
→ reference-manager
→ quality-assurance-auditor
→ workflow-orchestrator
```

Each stage should leave an artifact that the next stage can inspect.

## Stage-by-stage workflow

### 0. Global setup (run once)

**workflow-orchestrator** → Check current project state and decide the next skill.
**problem-parser** → Parse the problem into structured task description.
**problem-classifier** → Classify each subquestion by task type.
**related-paper-analyzer** → Analyze user-provided papers for transferable method ideas.
**symbol-table-builder** → Build unified global symbol table across all subquestions.
**model-assumptions-builder** → Extract and organize model assumptions.

### Per-question iteration loop

```
method-selector → data-auditor-cleaner → model-code-analyzer
  → code generators → code-reviewer → result-report-generator
  → [modeler reviews report → loop back to method-selector if needed]
  → [or proceed to final method lock-in]
→ robustness-checker → final-method-explainer
  → figure-table-planner → math-figure-generator
  → solution-package-builder → paper-section-writer
  → paper-polisher → reference-manager
```

### Final assembly

**quality-assurance-auditor** → Final QA check.
**workflow-orchestrator** → Final assembly if QA passes.

## Delivery chain per subquestion

For each Qx, the full chain is:

1. `method-selector` → `methods/Qx/qx_method_candidates.md`
2. `data-auditor-cleaner` → cleaned data + data report
3. `model-code-analyzer` → `code/model-code-analyzer.md`
4. `python/matlab-model-code-generator` → `code/Qx/*` or `code/matlab/Qx/*`
5. `code-reviewer` → reviewed code
6. `result-report-generator` → `results/Qx/experiments/roundN/qx_experiment_report_roundN.md`
7. [Loop 1-6 until method converges; modeler records in `methods/Qx/qx_method_iteration_log.md`]
8. `robustness-checker` → `robustness/Qx/qx_robustness_report.md`
9. `final-method-explainer` → `methods/Qx/qx_final_method_explanation.md`
10. `result-report-generator` (final mode) → `results/Qx/reports/qx_final_result_analysis.md`
11. `figure-table-planner` → `methods/Qx/qx_figure_table_plan.md`
12. `math-figure-generator` → paper figures
13. `solution-package-builder` → `results/Qx/reports/qx_solution_package_for_writer.md`
14. `paper-section-writer` → `paper/sections/qx.tex`
15. `paper-polisher` → polished paper sections
16. `reference-manager` → `paper/refs.bib` + reference audit
17. Modeler review + Programmer review → Finalized

## Workflow gates

These gates are intentional:

- No method selection before problem parsing.
- No code generation before candidate method pool exists.
- No paper claims before result artifacts exist.
- No paper writing for Qx before `qx_final_method_explanation.md` exists (Rule 1).
- No writer handoff for Qx before `qx_final_result_analysis.md` exists (Rule 2).
- No writer proceeds for Qx before `qx_solution_package_for_writer.md` exists (Rule 3).
- No model superiority claim without baseline and robustness evidence.
- No final assembly before QA passes.
- Raw data must remain unchanged.

## Three critical rules

Rule 1: No final paper writing without final method explanation.
Rule 2: No writer handoff without final result analysis.
Rule 3: Writer reads from solution package, not scattered results.

## Workspace structure

```
project/
├── planning/                   # Global planning
│   ├── parse/                  # Problem parse outputs
│   ├── classification/         # Problem classification outputs
│   ├── symbol_table.md         # Unified symbol table
│   ├── model_assumptions.md    # Global model assumptions
│   ├── question_dependency.md  # Subquestion dependency map
│   └── progress_dashboard.md   # Live progress tracking
├── methods/Qx/                 # Method artifacts per subquestion
├── code/Qx/                    # Python code
├── code/matlab/Qx/             # MATLAB code
├── results/Qx/
│   ├── experiments/roundN/     # Experiment outputs
│   └── reports/                # Analysis reports
├── robustness/Qx/              # Robustness reports
├── paper/                      # Paper materials
└── workspace/                  # Data and external workspace
```

## When to go backward

Going backward is normal. Use the earliest skill that can fix the issue.

| Problem | Go back to |
|---------|-----------|
| Problem misunderstood | `problem-parser` |
| Task type wrong | `problem-classifier` |
| Method data mismatch | `method-selector` |
| Symbol conflict across Qs | `symbol-table-builder` |
| Assumption conflict | `model-assumptions-builder` |
| Data issues | `data-auditor-cleaner` |
| Code fails or mismatches method | `code-reviewer` |
| Experiment results poor | `method-selector` (revise methods) |
| Conclusions unstable | `robustness-checker` |
| Figure has no evidence | `figure-table-planner` |
| Paper claim unsupported | `paper-section-writer` |
| Wording issues | `paper-polisher` |
| Citation unverified | `reference-manager` |
| Final blockers | `quality-assurance-auditor` |

## Notes

This workflow does not replace modeling judgment. It is a guardrail.

You can skip or merge stages when there is a good reason, but the choice should be explicit. If a result, figure, or claim cannot be traced back to an artifact, treat it as unfinished.

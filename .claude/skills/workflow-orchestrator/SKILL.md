---
name: workflow-orchestrator
description: Manage the full mathematical modeling contest workflow and decide which skill should be used next.
license: MIT
---

# Purpose

Coordinate the full mathematical modeling contest workflow from problem intake to final QA.

This skill does not solve models, write code, generate figures, or draft paper sections directly. Its role is to inspect the current project state, identify missing or blocked artifacts, enforce workflow gates, and decide the next appropriate skill to use.

# When to use

Use this skill:

- At the beginning of a new mathematical modeling contest task.
- After any major artifact is created, updated, or reviewed.
- When the user is unsure what to do next.
- Before generating code, writing paper text, assembling the final paper, or submitting results.
- When the workflow appears inconsistent, incomplete, or out of order.

# Preconditions

This skill can run with minimal context. At least one of the following should be available:

- A contest problem statement.
- A workspace directory.
- Existing workflow artifacts.
- A user-provided status update.
- A request to determine the next step.

If none of these exists, ask the user to provide the problem statement, workspace tree, or current progress summary.

# Inputs

Inspect or request the following, depending on availability:

- Problem statement or problem file location.
- Current workspace tree.
- Existing artifacts under:
  - `workspace/problem/`
  - `workspace/data_raw/`
  - `workspace/data_clean/`
  - `workspace/scripts/`
  - `workspace/results/`
  - `workspace/figures/`
  - `workspace/paper_sections/`
  - `workspace/final_paper/`
- Existing parsed problem, classification, related paper analysis, method plan, data report, model results, robustness report, figure plan, paper sections, or QA report.
- User constraints, such as contest deadline, allowed tools, preferred language, or team role distribution.

# Workflow

1. Inspect the current task state.
   - Identify available files, user-provided context, and completed artifacts.
   - Do not assume that a stage is complete only because a later file exists.

2. Map the task to the standard workflow stages:
   - problem intake
   - problem parsing
   - problem classification
   - related paper analysis
   - method selection
   - data auditing and cleaning
   - model code generation
   - code review
   - robustness and sensitivity checking
   - figure and table planning
   - paper section writing
   - quality assurance
   - final assembly

3. Enforce workflow gates.
   - Do not allow method selection before problem parsing.
   - Do not allow code generation before a validated method plan exists.
   - Do not allow paper claims before model results exist.
   - Do not allow final assembly before QA passes.

4. Identify missing or blocked artifacts.
   - Mark missing files explicitly.
   - Mark blocked steps separately from merely incomplete steps.
   - If the blocker is user-side information, state exactly what is needed.

5. Select the next skill.
   - Choose exactly one primary next skill.
   - Provide a short reason for the choice.
   - Provide concrete next actions for the user or agent.

6. Return a workflow state report.
   - Keep the report concise, structured, and actionable.
   - Do not perform the downstream skill's work inside this skill.

# Outputs

Produce a workflow state report containing:

- `current_stage`
- `completed_artifacts`
- `missing_artifacts`
- `blocked_items`
- `workflow_gates`
- `next_skill`
- `next_actions`
- `handoff_context`

# Output format

Prefer this JSON-compatible structure:

```json
{
  "current_stage": "problem_parsing",
  "completed_artifacts": {
    "problem_statement": true,
    "problem_parse": false,
    "problem_classification": false,
    "related_paper_analysis": false,
    "method_plan": false,
    "data_report": false,
    "cleaned_data": false,
    "model_scripts": false,
    "model_results": false,
    "robustness_report": false,
    "figure_plan": false,
    "paper_sections": false,
    "qa_report": false
  },
  "missing_artifacts": [
    "workspace/problem/problem_parse.json"
  ],
  "blocked_items": [],
  "workflow_gates": {
    "method_selection_allowed": false,
    "code_generation_allowed": false,
    "paper_writing_allowed": false,
    "final_assembly_allowed": false
  },
  "next_skill": "problem-parser",
  "next_actions": [
    "Parse the contest problem into goals, objects, constraints, data, outputs, and subquestions.",
    "Save the parsed result under workspace/problem/."
  ],
  "handoff_context": {
    "reason": "The problem has not been parsed yet, so classification and method selection are blocked.",
    "required_inputs_for_next_skill": [
      "problem statement",
      "attachment descriptions if available"
    ]
  }
}
```

If a JSON block is too rigid for the situation, use a concise Markdown report with the same fields.

# Rules

- Do not solve the mathematical model.
- Do not select specific algorithms unless the next step is only to route to `method-selector`.
- Do not write code.
- Do not draft paper sections.
- Do not fabricate missing artifacts.
- Do not infer completion without evidence.
- Do not skip workflow gates for speed.
- Do not approve final delivery without QA.
- Keep recommendations tied to traceable artifacts.
- Prefer one clear next step over many speculative branches.

# Verification

Before handing off, verify:

- The chosen `next_skill` matches the earliest incomplete required stage.
- No workflow gate is bypassed.
- Missing artifacts are explicitly listed.
- Blocked items are separated from ordinary incomplete items.
- The next actions are concrete enough to execute.
- The handoff context includes the minimum inputs needed by the next skill.

# Failure modes

Stop and report a blocker if:

- The problem statement is missing.
- The user asks to generate code but no method plan exists.
- The user asks to write numerical conclusions but no model results exist.
- The user asks to assemble or submit the final paper before QA passes.
- The workspace state is inconsistent, such as paper claims existing without corresponding results.
- Multiple incompatible interpretations of the current task exist and the choice affects workflow order.

# Stop conditions

This skill must stop instead of guessing when:

- Required artifacts cannot be located.
- The current stage cannot be determined from available context.
- A downstream skill would need fabricated data, fabricated results, or unsupported assumptions.
- The user request conflicts with the workflow gates.

When stopping, output:

- the blocker
- why it matters
- the minimum information or artifact needed to continue
- the recommended next skill once the blocker is resolved

# Handoff

Typical handoff order:

1. `problem-parser`
2. `problem-classifier`
3. `related-paper-analyzer`
4. `method-selector`
5. `data-auditor-cleaner`
6. `model-code-generator`
7. `code-reviewer`
8. `robustness-checker`
9. `figure-table-planner`
10. `paper-section-writer`
11. `quality-assurance-auditor`
12. back to `workflow-orchestrator`

If the workflow is complete and QA has passed, hand off to final assembly or user review.

# Examples

## Example 1: New contest problem

Input state:

- Problem PDF exists in `workspace/problem/`.
- No parsed problem exists.

Output:

```json
{
  "current_stage": "problem_parsing",
  "next_skill": "problem-parser",
  "next_actions": [
    "Extract goals, objects, constraints, data, outputs, and subquestions.",
    "Save the parsed artifact under workspace/problem/."
  ]
}
```

## Example 2: User asks to generate code too early

Input state:

- Problem has been parsed.
- Problem has not been classified.
- Method plan does not exist.
- User asks for modeling code.

Output:

```json
{
  "current_stage": "problem_classification",
  "blocked_items": [
    "Code generation is blocked because no validated method plan exists."
  ],
  "next_skill": "problem-classifier",
  "next_actions": [
    "Classify each subquestion before literature analysis and method selection.",
    "Then hand off to related-paper-analyzer."
  ]
}
```

## Example 3: User asks to assemble final paper before QA

Input state:

- Paper sections exist.
- QA report does not exist.

Output:

```json
{
  "current_stage": "quality_assurance",
  "blocked_items": [
    "Final assembly is blocked because QA has not passed."
  ],
  "next_skill": "quality-assurance-auditor",
  "next_actions": [
    "Audit problem coverage, model-result consistency, figure references, robustness checks, and unsupported claims."
  ]
}
```

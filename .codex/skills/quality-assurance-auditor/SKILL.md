---
name: quality-assurance-auditor
description: Audit the complete mathematical modeling solution before final submission and identify blocking issues.
license: MIT
---

# Purpose

Audit the complete mathematical modeling contest solution before final submission.

This skill checks whether the solution answers every subquestion, whether models, code, results, figures, robustness checks, and paper sections are mutually consistent, and whether unsupported claims or missing artifacts block final delivery.

This skill does not invent missing results, rewrite the full paper, rerun models, or approve submission when evidence is incomplete.

# When to use

Use this skill:

- After `paper-section-writer` has drafted paper sections.
- Before final assembly or submission.
- When checking whether the solution is logically complete.
- When identifying blocking issues, minor issues, and repair actions.
- When deciding whether to return to an upstream skill.

# Preconditions

The following should already exist or be provided:

- Parsed problem and subquestions.
- Problem classification.
- Method plan.
- Data report and cleaned data summary, if data is used.
- Reviewed model code.
- Model result artifacts.
- Robustness or sensitivity report.
- Figure-table plan.
- Paper section drafts.

If paper sections are missing, hand back to `paper-section-writer`.

If model results are missing, hand back to `model-code-generator` or `code-reviewer`.

# Inputs

Use or request:

- `workspace/problem/problem_parse.json`
- `workspace/problem/problem_classification.json`
- `workspace/problem/method_plan.json`
- `workspace/results/data_report.md`
- result files under `workspace/results/`
- robustness reports and sensitivity tables under `workspace/results/`
- figures under `workspace/figures/`
- paper drafts under `workspace/paper_sections/`
- generated scripts under `workspace/scripts/`
- contest requirements or submission rules if available

# Workflow

1. Check problem coverage.
   - Verify every original subquestion is represented.
   - Verify each subquestion has a corresponding model, result, and written answer.
   - Mark unanswered or partially answered subquestions as blocking issues.

2. Check artifact lineage.
   - Trace each major paper claim back to problem parse, method plan, data, code, result, figure, or robustness artifact.
   - Mark claims without artifacts as unsupported.

3. Check model consistency.
   - Verify models match the classified problem types.
   - Verify baseline models exist for main models unless explicitly justified.
   - Verify the paper does not describe a different model than the code or method plan.

4. Check data and result consistency.
   - Verify raw data was not overwritten.
   - Verify cleaned data and field mappings are documented.
   - Verify result tables come from reviewed scripts or accepted computation.
   - Verify units, variables, and dimensions are consistent.

5. Check robustness evidence.
   - Verify robustness or sensitivity checks exist before final delivery.
   - Verify robustness claims do not exceed completed checks.
   - Separate stable and fragile conclusions.

6. Check figures and tables.
   - Verify every referenced figure or table exists or is explicitly marked as planned.
   - Verify every figure or table has a source artifact.
   - Verify captions support the intended claim.
   - Reject decorative or unsupported visuals.

7. Check paper quality.
   - Verify assumptions are necessary and linked to modeling needs.
   - Verify notation is defined before use.
   - Verify conclusions map back to subquestions.
   - Verify limitations are not hidden.
   - Verify no references, numerical values, experiments, or conclusions are fabricated.

8. Produce a QA report.
   - Separate blocking issues from minor issues.
   - Provide a repair plan.
   - Route each repair to the appropriate upstream skill.
   - Approve final assembly only if no blocking issues remain.

# Outputs

Produce a QA report containing:

- `qa_status`
- `blocking_issues`
- `minor_issues`
- `unsupported_claims`
- `artifact_traceability`
- `subquestion_coverage`
- `repair_plan`
- `recommended_next_skill`
- `final_assembly_allowed`

# Output format

Prefer this JSON-compatible structure:

```json
{
  "qa_status": "failed",
  "final_assembly_allowed": false,
  "blocking_issues": [
    {
      "issue": "Q3 has no validated result artifact.",
      "why_it_matters": "The final conclusion cannot answer all subquestions.",
      "repair_skill": "model-code-generator"
    }
  ],
  "minor_issues": [
    {
      "issue": "Figure caption does not explain the main takeaway.",
      "repair_skill": "paper-section-writer"
    }
  ],
  "unsupported_claims": [
    {
      "claim": "The model is globally stable.",
      "reason": "Robustness checks only cover limited perturbations."
    }
  ],
  "subquestion_coverage": [
    {
      "id": "Q1",
      "status": "answered",
      "evidence": [
        "method plan",
        "result table",
        "paper section"
      ]
    }
  ],
  "repair_plan": [
    "Generate or provide Q3 result artifact.",
    "Revise unsupported robustness claim.",
    "Update figure caption."
  ],
  "recommended_next_skill": "workflow-orchestrator"
}
```

If JSON is too rigid, use a concise Markdown report with the same fields.

# QA checklist

Check at least:

- every subquestion is answered
- every answer has model-result-paper alignment
- every main model has a baseline or justified exception
- every numerical claim has a result artifact
- every figure or table has a source artifact
- every robustness claim has robustness evidence
- every conclusion maps to a subquestion
- assumptions are necessary and stated clearly
- notation is defined and consistent
- code outputs match paper claims
- raw data is preserved
- limitations are stated
- no fabricated data, references, results, figures, or experiments exist

# Rules

- Do not fabricate missing evidence.
- Do not approve final assembly with blocking issues.
- Do not hide uncertainty.
- Do not treat planned artifacts as completed artifacts.
- Do not rewrite the full paper inside QA.
- Do not rerun models unless explicitly asked by the user.
- Do not change upstream artifacts silently.
- Route repairs to the correct upstream skill.
- Keep QA findings specific and actionable.

# Verification

Before approving final assembly, verify:

- `blocking_issues` is empty.
- `unsupported_claims` is empty or explicitly removed from final text.
- every subquestion is covered.
- baseline and robustness requirements are satisfied.
- all figures and tables are traceable.
- final conclusions do not exceed evidence.
- `final_assembly_allowed` is true only when QA passes.

# Failure modes

Stop and report a blocker if:

- paper sections are missing
- model results are missing
- robustness evidence is missing before final delivery
- figures or tables are referenced but not available
- claims cannot be traced to artifacts
- subquestions are unanswered
- final assembly is requested before QA passes

# Stop conditions

This skill must stop instead of guessing when:

- an issue requires missing data or missing result artifacts
- the paper and code describe incompatible models
- final conclusions depend on unsupported claims
- fixing the issue requires upstream modeling, code, or data work
- continuing would require inventing evidence

When stopping, output:

- blocker
- why it matters
- affected section or subquestion
- required artifact
- recommended repair skill
- next action

# Handoff

If QA passes, hand off to:

`workflow-orchestrator`

with `final_assembly_allowed: true`.

If QA fails, hand off to the relevant upstream skill:

- `problem-parser` for missing or misread subquestions
- `problem-classifier` for wrong task type mapping
- `method-selector` for model-plan issues
- `data-auditor-cleaner` for data issues
- `model-code-generator` for missing result generation
- `code-reviewer` for code or artifact issues
- `robustness-checker` for missing stability evidence
- `figure-table-planner` for unsupported visuals
- `paper-section-writer` for writing or traceability issues

# Examples

## Example 1: QA fails because a subquestion is unanswered

```json
{
  "qa_status": "failed",
  "final_assembly_allowed": false,
  "blocking_issues": [
    {
      "issue": "Q3 is not answered in the paper.",
      "why_it_matters": "Final submission must answer every required subquestion.",
      "repair_skill": "workflow-orchestrator"
    }
  ],
  "recommended_next_skill": "workflow-orchestrator"
}
```

## Example 2: QA fails because a claim is unsupported

```json
{
  "qa_status": "failed",
  "final_assembly_allowed": false,
  "unsupported_claims": [
    {
      "claim": "The proposed prediction model is highly accurate.",
      "reason": "No error metric or baseline comparison is provided.",
      "repair_skill": "robustness-checker"
    }
  ],
  "recommended_next_skill": "robustness-checker"
}
```

## Example 3: QA passes

```json
{
  "qa_status": "passed",
  "final_assembly_allowed": true,
  "blocking_issues": [],
  "minor_issues": [],
  "recommended_next_skill": "workflow-orchestrator"
}
```

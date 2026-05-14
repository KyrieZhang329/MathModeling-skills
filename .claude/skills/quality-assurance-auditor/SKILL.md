---
name: quality-assurance-auditor
description: Audit the complete mathematical modeling solution before final submission and identify blocking issues.
license: MIT
---

# Purpose

Audit the complete mathematical modeling contest solution before final submission.

This skill checks whether the solution answers every subquestion, whether models, code, results, figures, robustness checks, and paper sections are mutually consistent, whether the full workflow completeness requirements are met, and whether unsupported claims or missing artifacts block final delivery.

This skill checks BOTH paper quality AND workflow completeness. It verifies that every subquestion has gone through the full delivery chain: candidate methods → experiments → final method explanation → final result analysis → solution package → paper sections.

This skill does not invent missing results, rewrite the full paper, rerun models, or approve submission when evidence is incomplete.

# When to use

Use this skill:

- After `paper-section-writer` has drafted paper sections for all subquestions.
- Before final assembly or submission.
- When checking whether the solution is logically complete.
- When identifying blocking issues, minor issues, and repair actions.
- When deciding whether to return to an upstream skill.

# Preconditions

The following should already exist or be provided:

- Parsed problem and subquestions.
- Problem classification.
- Candidate method pools per subquestion.
- Data report and cleaned data summary, if data is used.
- Reviewed model code.
- Model result artifacts per subquestion.
- Robustness or sensitivity reports per subquestion.
- Figure-table plans per subquestion.
- Paper section drafts for all subquestions.

If paper sections are missing, hand back to `paper-section-writer`.

If model results are missing, hand back to `python-model-code-generator`, `matlab-model-code-generator`, or `code-reviewer`.

# Inputs

Use or request:

- `workspace/problem/problem-parser/problem_parse.json`
- `workspace/problem/problem-classifier/problem_classification.json`
- `methods/Qx/qx_method_candidates.md` (all subquestions)
- `methods/Qx/qx_method_iteration_log.md` (all subquestions)
- `methods/Qx/qx_final_method_explanation.md` (all subquestions)
- `results/Qx/reports/qx_final_result_analysis.md` (all subquestions)
- `results/Qx/reports/qx_solution_package_for_writer.md` (all subquestions)
- Result files under `results/Qx/experiments/`
- Robustness reports under `robustness/Qx/`
- Figure-table plans under `methods/Qx/`
- Figures under `results/Qx/experiments/*/figures/` and `paper/figures/`
- Paper drafts under `paper/sections/`
- Generated code under `code/`
- `planning/progress_dashboard.md` (workflow status)
- Contest requirements or submission rules if available

# Workflow

1. Check problem coverage.
   - Verify every original subquestion is represented in the paper.
   - Verify each subquestion has a corresponding model, result, and written answer.
   - Mark unanswered or partially answered subquestions as blocking issues.

2. **Check workflow completeness (NEW — expanded check).**
   - For each subquestion, verify the full delivery chain:
     1. Candidate method pool exists (`methods/Qx/qx_method_candidates.md`).
     2. Experiment reports exist (at least one round at `results/Qx/experiments/roundN/`).
     3. Method iteration log exists (`methods/Qx/qx_method_iteration_log.md`).
     4. Final method explanation exists (`methods/Qx/qx_final_method_explanation.md`).
     5. Final result analysis exists (`results/Qx/reports/qx_final_result_analysis.md`).
     6. Solution package exists (`results/Qx/reports/qx_solution_package_for_writer.md`).
     7. Robustness report exists (`robustness/Qx/qx_robustness_report.md`) or a justified exception.
     8. Figure-table plan exists (`methods/Qx/qx_figure_table_plan.md`).
   - Mark any missing link in this chain as a blocking issue.

3. Check the three critical rules.
   - Rule 1: Verify paper model description matches the final method explanation (not an early candidate pool).
   - Rule 2: Verify paper results come from the final result analysis (not raw experiment outputs directly).
   - Rule 3: Verify the solution package exists and the paper writer used it.

4. Check artifact lineage.
   - Trace each major paper claim back to problem parse, method explanation, data, code, result, figure, or robustness artifact.
   - Mark claims without artifacts as unsupported.

5. Check model consistency.
   - Verify models match the classified problem types.
   - Verify baseline models exist for main models unless explicitly justified.
   - Verify the paper describes the final method (from `qx_final_method_explanation.md`), not an earlier candidate.
   - Verify eliminated methods are mentioned appropriately.

6. Check data and result consistency.
   - Verify raw data was not overwritten.
   - Verify cleaned data and field mappings are documented.
   - Verify result tables come from reviewed scripts or accepted computation.
   - Verify units, variables, and dimensions are consistent.

7. Check paper quality.
   - Verify every referenced figure or table exists on disk.
   - Verify figures in the paper match the figure-table plan's Type 3 (论文图) classification.
   - Verify no Type 1 (诊断图) figures accidentally appear in the paper.
   - Verify every figure or table has a source artifact.
   - Verify captions support the intended claim.
   - Verify assumptions are necessary and linked to modeling needs.
   - Verify notation is defined before use and consistent with the global symbol table.
   - Verify conclusions map back to subquestions.
   - Verify limitations are not hidden.
   - Reject decorative or unsupported visuals.

8. Check for fabrication and exaggeration.
   - Verify no references are fabricated.
   - Verify no numerical values are invented.
   - Verify no experiments are claimed that were not run.
   - Verify robustness claims do not exceed completed checks.
   - Verify conclusions do not exceed evidence.
   - Flag any claim that cannot be traced to an artifact.

9. Produce a QA report.
   - Save to `paper/qa_report.md`.
   - Separate blocking issues from minor issues.
   - Provide a repair plan.
   - Route each repair to the appropriate upstream skill.
   - Approve final assembly only if no blocking issues remain.

# Outputs

Produce a QA report at:

- `paper/qa_report.md`

Containing:

- `qa_status` (passed / failed)
- `final_assembly_allowed` (true / false)
- `blocking_issues`
- `minor_issues`
- `workflow_completeness_check` (per subquestion)
- `three_critical_rules_check`
- `unsupported_claims`
- `fabrication_risks`
- `artifact_traceability`
- `subquestion_coverage`
- `repair_plan`
- `recommended_next_skill`

# Output format

```markdown
# QA Report

## Overall Status

- **QA Status**: passed / failed
- **Final Assembly Allowed**: true / false
- **Date**: [timestamp]

## Workflow Completeness Check

| Check | Q1 | Q2 | Q3 | Q4 |
|-------|----|----|----|-----|
| 1. Candidate method pool | ✅ | ✅ | ✅ | ❌ |
| 2. Experiment report(s) | ✅ (R2) | ✅ (R1) | ❌ | ❌ |
| 3. Method iteration log | ✅ | ❌ | ❌ | ❌ |
| 4. Final method explanation | ✅ | ❌ | ❌ | ❌ |
| 5. Final result analysis | ✅ | ❌ | ❌ | ❌ |
| 6. Solution package | ✅ | ❌ | ❌ | ❌ |
| 7. Robustness report | ✅ | ❌ | ❌ | ❌ |
| 8. Figure-table plan | ✅ | ❌ | ❌ | ❌ |

## Three Critical Rules Check

| Rule | Q1 | Q2 | Q3 | Q4 | Status |
|------|----|----|----|-----|--------|
| Rule 1: Paper method matches final method explanation | ✅ | ❌ (no final explanation) | N/A | N/A | Q2 FAIL |
| Rule 2: Paper results from final result analysis | ✅ | N/A | N/A | N/A | PASS |
| Rule 3: Solution package exists and was used | ✅ | ❌ (no package) | N/A | N/A | Q2 FAIL |

## Subquestion Coverage

| Subquestion | Answered in Paper | Model Exists | Results Exist | Robustness Exists | Status |
|-------------|-------------------|-------------|---------------|-------------------|--------|
| Q1 | ✅ | ✅ (entropy-TOPSIS) | ✅ | ✅ | Complete |
| Q2 | ⏳ (partial draft) | ❌ (no final) | ❌ | ❌ | Blocked |
| Q3 | ❌ | ❌ | ❌ | ❌ | Blocked |
| Q4 | ❌ | ❌ | ❌ | ❌ | Blocked |

## Blocking Issues

| # | Issue | Affects | Why It Matters | Repair Skill |
|---|-------|---------|---------------|--------------|
| 1 | Q2 has no final method explanation | Q2 completion | Paper cannot describe the final model | `final-method-explainer` |
| 2 | Q3 not started | Q3 completion | Subquestion unanswered | `workflow-orchestrator` |
| 3 | Q4 not started | Q4 completion | Subquestion unanswered | `workflow-orchestrator` |

## Minor Issues

| # | Issue | Affects | Repair Skill |
|---|-------|---------|--------------|
| 1 | Q1 Figure 1.3 caption does not state the main takeaway | Q1 paper section | `paper-section-writer` |
| 2 | Q1 robustness report missing boundary condition statement | Q1 robustness section | `robustness-checker` |

## Unsupported Claims

| # | Claim | Location | Why Unsupported | Action |
|---|-------|----------|----------------|--------|
| 1 | "The model is globally stable." | paper/sections/q1.tex | Robustness only checked ±10% weight perturbation | Downgrade to "stable under moderate perturbation" |

## Fabrication Risks

| # | Risk | Location | Evidence |
|---|------|----------|----------|
| — | (none detected) | — | — |

## Artifact Traceability (spot check)

| Paper Claim | Source Artifact | Traceable? |
|-------------|----------------|------------|
| "City A ranks highest with score 0.88." | `results/Q1/experiments/final/tables/m2_scores.csv` | ✅ |

## Repair Plan

1. **BLOCKING**: Run `final-method-explainer` for Q2 to produce `methods/Q2/q2_final_method_explanation.md`.
2. **BLOCKING**: Run `workflow-orchestrator` to plan Q3 and Q4 execution.
3. **MINOR**: Update Q1 Figure 1.3 caption via `paper-section-writer`.
4. **MINOR**: Add boundary condition statement to Q1 robustness report via `robustness-checker`.

## Recommended Next Skill

- **If QA failed**: `workflow-orchestrator` (with repair plan)
- **If QA passed**: `workflow-orchestrator` (for final assembly)
```

# Expanded QA checklist

## Workflow completeness (per subquestion)
- [ ] Candidate method pool exists
- [ ] Experiment report(s) exist
- [ ] Method iteration log exists
- [ ] Final method explanation exists
- [ ] Final result analysis exists
- [ ] Solution package exists
- [ ] Robustness report exists (or justified exception)
- [ ] Figure-table plan exists

## Three critical rules
- [ ] Paper model description matches final method explanation (not earlier candidate pool)
- [ ] Paper results come from final result analysis (not raw experiment outputs)
- [ ] Solution package exists and was used as primary source

## Problem coverage
- [ ] Every subquestion is answered in the paper
- [ ] Every answer has model-result-paper alignment

## Model consistency
- [ ] Every main model has a baseline or justified exception
- [ ] Paper describes the final method, not an early draft
- [ ] Eliminated methods are mentioned where appropriate

## Evidence quality
- [ ] Every numerical claim has a result artifact
- [ ] Every figure or table has a source artifact
- [ ] Every figure in the paper exists on disk
- [ ] No Type 1 (诊断图) figures in the paper
- [ ] Every robustness claim has robustness evidence

## Paper quality
- [ ] Every conclusion maps to a subquestion
- [ ] Assumptions are necessary and stated clearly
- [ ] Notation is defined and consistent (cross-check with global symbol table)
- [ ] Code outputs match paper claims
- [ ] Raw data is preserved
- [ ] Limitations are stated

## Anti-fabrication
- [ ] No fabricated data, references, results, figures, or experiments
- [ ] No invented numerical values
- [ ] No unsupported superiority claims
- [ ] No hidden uncertainty

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
- Check workflow completeness — it's not just about paper quality.

# Verification

Before approving final assembly, verify:

- `blocking_issues` is empty.
- `workflow_completeness_check` passes for all subquestions (8/8 checks per subquestion, or justified exceptions).
- `three_critical_rules_check` passes for all subquestions.
- `unsupported_claims` is empty or explicitly removed from final text.
- Every subquestion is covered.
- Baseline and robustness requirements are satisfied.
- All figures and tables are traceable.
- Final conclusions do not exceed evidence.
- No fabrication risks detected.
- `final_assembly_allowed` is true only when QA passes.

# Failure modes

Stop and report a blocker if:

- Paper sections are missing.
- Model results are missing.
- Robustness evidence is missing before final delivery.
- Figures or tables are referenced but not available.
- Claims cannot be traced to artifacts.
- Subquestions are unanswered.
- Any critical rule is violated.
- Workflow completeness chain is broken.
- Final assembly is requested before QA passes.

# Stop conditions

This skill must stop instead of guessing when:

- An issue requires missing data or missing result artifacts.
- The paper and code describe incompatible models.
- Final conclusions depend on unsupported claims.
- Fixing the issue requires upstream modeling, code, or data work.
- Continuing would require inventing evidence.

When stopping, output:
- blocker
- why it matters
- affected section or subquestion
- required artifact
- recommended repair skill
- next action

# Handoff

If QA passes:
→ `workflow-orchestrator` with `final_assembly_allowed: true`.

If QA fails:
→ `workflow-orchestrator` with repair plan routing to the relevant upstream skill.

Repair routing:
- `problem-parser` → missing or misread subquestions
- `problem-classifier` → wrong task type mapping
- `method-selector` → missing candidate pool
- `data-auditor-cleaner` → data issues
- `model-code-analyzer` or code generators → missing code
- `code-reviewer` → code or artifact issues
- `result-report-generator` → missing experiment report or final result analysis
- `final-method-explainer` → missing final method explanation
- `solution-package-builder` → missing solution package
- `robustness-checker` → missing stability evidence
- `figure-table-planner` → unsupported visuals or missing figure plan
- `paper-section-writer` → writing or traceability issues

# Examples

## Example 1: QA fails — workflow incomplete (Q2 missing final method)

```json
{
  "qa_status": "failed",
  "final_assembly_allowed": false,
  "blocking_issues": [
    {
      "issue": "Q2 has no final method explanation (methods/Q2/q2_final_method_explanation.md missing).",
      "why_it_matters": "The paper cannot write the Q2 model construction section. Rule 1 violated.",
      "repair_skill": "final-method-explainer"
    }
  ],
  "workflow_completeness_check": {
    "Q1": {"passed": true, "missing": []},
    "Q2": {"passed": false, "missing": ["final_method_explanation", "final_result_analysis", "solution_package", "robustness_report"]},
    "Q3": {"passed": false, "missing": ["all"]},
    "Q4": {"passed": false, "missing": ["all"]}
  },
  "recommended_next_skill": "workflow-orchestrator"
}
```

## Example 2: QA fails — unsupported claim detected

```json
{
  "qa_status": "failed",
  "final_assembly_allowed": false,
  "unsupported_claims": [
    {
      "claim": "The prediction model is highly accurate.",
      "location": "paper/sections/q2.tex",
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
  "minor_issues": [
    {
      "issue": "Figure caption for Q1 Fig.1.3 could be more specific.",
      "repair_skill": "paper-section-writer"
    }
  ],
  "workflow_completeness_check": {
    "Q1": {"passed": true, "missing": []},
    "Q2": {"passed": true, "missing": []},
    "Q3": {"passed": true, "missing": []},
    "Q4": {"passed": true, "missing": []}
  },
  "three_critical_rules_check": {
    "Q1": {"all_passed": true},
    "Q2": {"all_passed": true},
    "Q3": {"all_passed": true},
    "Q4": {"all_passed": true}
  },
  "recommended_next_skill": "workflow-orchestrator"
}
```

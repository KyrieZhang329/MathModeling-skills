---
name: solution-package-builder
description: Integrate the modeler's final method explanation and the programmer's final result analysis into a single comprehensive writer-facing solution package that the paper writer can directly use to draft paper sections.
license: MIT
---

# Purpose

Build the writer-facing solution package that integrates all modeling and results work into one coherent, self-contained document. This is the single source of truth that the paper writer uses to draft paper sections for a subquestion.

This skill merges:
- The modeler's final method explanation (what we did and why).
- The programmer's final result analysis (what we got and what it means).
- Planned and generated figures and tables (what visual evidence exists).
- Robustness and sensitivity findings (how stable our conclusions are).

Into one structured package that the paper writer can read and directly translate into paper content without needing to dig through scattered artifacts or guess at missing information.

This skill does not write the actual paper sections, select models, run experiments, generate figures, or perform QA.

# When to use

Use this skill:

- After `final-method-explainer` has produced `methods/Qx/qx_final_method_explanation.md`.
- After `result-report-generator` has produced `results/Qx/reports/qx_final_result_analysis.md`.
- When both modeling and results work are locked in and the paper writer is waiting.
- When the user says: "prepare the solution package for Q1", "package everything for the writer", "build the writer's material package", "I need to write the paper section for Q2".
- Before `paper-section-writer`.
- When the workflow status for a subquestion is "Ready for Writer".

# Preconditions (CRITICAL — enforced as gates)

The following three files MUST exist before this skill can produce a complete package:

1. `methods/Qx/qx_final_method_explanation.md` — the modeler's definitive method document.
2. `results/Qx/reports/qx_final_result_analysis.md` — the programmer's definitive result document.
3. The actual figure and table files referenced in the result analysis (must exist on disk, not just be planned).

Additionally useful (but not strictly required):
- `results/Qx/experiments/final/` — final round outputs.
- `robustness/Qx/qx_robustness_report.md` — robustness and sensitivity analysis.
- Figure-table plan for this subquestion.

If `qx_final_method_explanation.md` is missing, stop and report: "This subquestion is not Ready for Writer — missing final method explanation. Hand back to the modeler or `final-method-explainer`."

If `qx_final_result_analysis.md` is missing, stop and report: "This subquestion is not Ready for Writer — missing final result analysis. Hand back to the programmer or `result-report-generator`."

If both exist but referenced figures/tables are missing, produce the package with warnings about missing visuals.

# Inputs

Use or request:

- `methods/Qx/qx_final_method_explanation.md` — REQUIRED.
- `results/Qx/reports/qx_final_result_analysis.md` — REQUIRED.
- `results/Qx/experiments/final/` — final experiment outputs directory.
- `robustness/Qx/qx_robustness_report.md` — robustness findings, if available.
- `methods/Qx/qx_method_candidates.md` — original candidate pool, for method selection narrative.
- `methods/Qx/qx_method_iteration_log.md` — iteration history, for demonstrating thoroughness.
- Figure-table plan relevant to this subquestion.
- Any existing paper draft sections for this subquestion (to check what's already been written).
- The problem parse for this subquestion's goals and constraints.

# Workflow

1. Verify prerequisites (gate check).
   - Confirm all three required files exist.
   - Verify referenced figures and tables exist on disk.
   - Read each input file thoroughly.
   - If any prerequisite is missing, stop and report exactly what is missing.

2. Extract and organize content.
   - From the final method explanation: extract the method name, selection justification, assumptions, symbols, mathematical model, solution procedure, and evaluation metrics.
   - From the final result analysis: extract key numerical results, figure interpretations, table interpretations, supported claims, and result limitations.
   - From the robustness report (if available): extract stable conclusions, fragile conclusions, perturbation results, and boundary conditions.
   - From the candidate pool (if available): extract the narrative of method exploration (what was tried, what was dropped, why).

3. Build the solution overview.
   - Write a concise summary of the subquestion goal and how it was solved.
   - State the final method in one sentence.
   - State the key result in one sentence.
   - State the confidence level.
   - This overview should be the first thing the paper writer reads.

4. Organize paper-ready content by section.
   - Map content to paper sections: Problem Analysis → Assumptions → Symbols → Model Construction → Model Solution → Results Analysis → Robustness → Conclusion.
   - For each section, extract the relevant content from the source documents.
   - Mark content as "ready to write as-is", "needs adaptation", or "for reference only".

5. Assign figures and tables to paper sections.
   - Every figure and table must be assigned to a specific paper section.
   - State what claim each visual supports.
   - Provide a draft caption for each.
   - Mark placement suggestions (top of section, after formula X, etc.).

6. Identify gaps and provide fallback guidance.
   - If a paper section has no supporting content, state what the writer should do (skip, mark as future work, write a brief note).
   - If robustness evidence is missing for a claim, downgrade the claim's confidence level.
   - If a figure exists but is not paper-quality, note what needs improvement.

7. Write the solution package.
   - Save as `results/Qx/reports/qx_solution_package_for_writer.md`.
   - Make it comprehensive but scannable — the writer should be able to find what they need quickly.
   - Include direct quotes from source documents where useful.
   - Include file paths for every referenced artifact.

8. Update the workflow status.
   - This subquestion is now "Ready for Writer" (confirmed, not just claimed).
   - Hand off to `paper-section-writer`.

# Outputs

Produce exactly one document:

- `results/Qx/reports/qx_solution_package_for_writer.md`

This is the ONLY document the paper writer should need to read for this subquestion (they can click through to source files for details).

# Output format

The solution package MUST contain all of the following sections:

```markdown
# Qx Solution Package for Paper Writer

> **Status**: Ready for Writer
> **Package built**: [date]
> **Sources**: [list of input files used]

---

## 0. Quick Reference

- **Subquestion**: [one-line description]
- **Final method**: [method name]
- **Key result**: [one-line main finding]
- **Confidence**: [high / medium / needs caution]
- **Estimated paper pages**: [rough estimate]

---

## 1. Subquestion Goal and Context

[2-3 sentences from problem parse: what this subquestion asks, what output is required, what constraints apply]

## 2. Method Overview

### 2.1 Final Method: [name]
[2-3 sentence summary from final method explanation]

### 2.2 Method Selection Narrative
[Brief story of exploration: what was tried, what was dropped, why the final method was chosen. Source: candidate pool + iteration log + final method explanation]

### 2.3 Relationship to Other Subquestions
[Inputs from / outputs to other subquestions]

---

## 3. Content for Paper Sections

### 3A. For "Model Construction" Section

**What to write** (source: final method explanation sections 2-4):
- [Extracted assumptions with justification — ready to write]
- [Symbol table excerpt — ready to write]
- [Mathematical model formulation — ready to write]
- [Solution procedure — ready to write]
- [Evaluation metrics — ready to write]

**Key formulas to include**:
1. [formula reference] — [why important]
2. [formula reference] — [why important]

**Writer notes**:
- [Any adaptation needed, tone suggestions, things to emphasize or avoid]

### 3B. For "Results Analysis" Section

**What to write** (source: final result analysis sections 2-4):
- [Key numerical results — ready to write]
- [Result interpretation — ready to write]
- [Baseline comparison — ready to write]

**Writer notes**:
- [Confidence qualifiers, unsupported claims to avoid]

### 3C. For "Robustness and Sensitivity" Section

**What to write** (source: robustness report):
- [Stable conclusions — ready to write]
- [Fragile conclusions — ready to write]
- [Perturbation findings — ready to write]

**If robustness report is missing**:
- [What the writer should say instead, e.g., mark as planned or note limitation]

### 3D. For "Conclusion" (per-subquestion portion)

**What to write**:
- [Direct answer to subquestion — ready to write]
- [Key numbers to cite — ready to write]
- [Limitations to acknowledge — ready to write]

---

## 4. Figure and Table Assignments

### Figures

| Figure File | Paper Section | Claim Supported | Draft Caption | Status |
|------------|---------------|-----------------|---------------|--------|
| [path] | [section] | [what claim] | [draft caption text] | exists / needs revision |

### Tables

| Table File | Paper Section | Claim Supported | Draft Caption | Status |
|-----------|---------------|-----------------|---------------|--------|
| [path] | [section] | [what claim] | [draft caption text] | exists / needs revision |

---

## 5. Supported Claims Inventory

### Claims Ready for Paper (with evidence)
| # | Claim | Evidence | Confidence |
|---|-------|----------|------------|
| 1 | [claim] | [figure/table/metric] | high / medium |

### Claims to Downgrade or Avoid
| # | Claim | Why Not Fully Supported | Suggested Alternative |
|---|-------|------------------------|----------------------|
| 1 | [overclaim] | [missing evidence] | [safer claim] |

### Claims from Eliminated Methods (for demonstrating thoroughness)
| # | Method | Finding | Can Mention in Paper? |
|---|--------|---------|----------------------|
| 1 | [eliminated method] | [what we learned] | yes / no / only briefly |

---

## 6. Paper Writing Sequence Suggestion

1. Start with [section] — it has the most complete content.
2. Then write [section] — refer to figures [X, Y].
3. ...
4. Save [section] for last — it depends on [other subquestion/QA].

---

## 7. Cross-References

- Final method explanation: `methods/Qx/qx_final_method_explanation.md`
- Final result analysis: `results/Qx/reports/qx_final_result_analysis.md`
- Robustness report: `robustness/Qx/qx_robustness_report.md` (or: NOT YET AVAILABLE)
- Candidate methods: `methods/Qx/qx_method_candidates.md`
- Iteration log: `methods/Qx/qx_method_iteration_log.md`
- Experiment outputs: `results/Qx/experiments/final/`
- Global symbol table: `planning/symbol_table.md` (if exists)

---

## 8. Package Completeness Checklist

- [x] Method explained
- [x] Results analyzed
- [x] Figures and tables assigned
- [x] Claims inventoried
- [x] Robustness addressed (or flagged as missing)
- [x] Paper section mapping complete
- [ ] Writer has all needed information — [YES / NO, with explanation]
```

# Rules

- CRITICAL: Do not produce this package if `qx_final_method_explanation.md` or `qx_final_result_analysis.md` is missing. Stop and report the gap.
- Do not write the actual paper sections. This is a package for the writer, not the paper itself.
- Every figure and table must be verified to exist on disk before being listed as "exists".
- Claims must be traceable to specific evidence (figure, table, metric, robustness check).
- Do not fabricate claims, evidence, figure interpretations, or numerical results.
- Do not hide missing evidence — if robustness is missing, flag it.
- If a source document contains contradictory information, flag the contradiction rather than choosing one side silently.
- Keep the recommendation voice neutral — this package informs the writer, it does not command them.
- Mark content quality levels: "ready to write as-is", "needs adaptation", "for reference only".
- If the package reveals that the subquestion is NOT actually Ready for Writer (e.g., key results are missing, major claims are unsupported), report this honestly and recommend returning to the appropriate upstream skill.

# Verification

Before handing off, verify:

- Both required input files exist and have been read.
- Every referenced figure and table file exists on disk (or is explicitly marked as missing).
- All major content categories are covered: method, results, figures/tables, claims, robustness.
- Paper section mapping is complete.
- The quick reference (section 0) is accurate and self-contained.
- Cross-reference file paths are correct.
- No fabricated content exists.
- The package completeness checklist is honest.

# Failure modes

Stop and report a blocker if:

- `qx_final_method_explanation.md` is missing — the subquestion is not Ready for Writer.
- `qx_final_result_analysis.md` is missing — the subquestion is not Ready for Writer.
- Both files exist but directly contradict each other (e.g., method explanation says "entropy weights" but result analysis says "equal weights").
- All referenced figures and tables are missing — there is no visual evidence.
- The package reveals that major claims in the result analysis are unsupported by actual data.
- A required input file is corrupted or unreadable.

# Stop conditions

This skill must stop instead of guessing when:

- Required prerequisites are missing (see above).
- Source documents contradict each other on material facts and the contradiction cannot be resolved from available information.
- A figure or table is referenced but does not exist, and the gap is material to the paper.
- The content would require inventing bridging narrative between contradictory sources.
- The package would mislead the writer into making unsupported claims.

When stopping, output:

- the blocker
- which prerequisite or artifact is missing/problematic
- what the writer CAN still write (partial content)
- what must be resolved before the package can be completed
- recommended next skill (usually `workflow-orchestrator`, `final-method-explainer`, or `result-report-generator`)

# Handoff

After producing `results/Qx/reports/qx_solution_package_for_writer.md`, hand off to:

`paper-section-writer`
— with the solution package path, figure/table inventory, claims inventory, and section mapping.

If the package reveals missing prerequisites, hand back to:

`workflow-orchestrator`
— with specific gap report and recommendation for which upstream skill to invoke.

Do not hand off to `quality-assurance-auditor` directly — QA should run after paper sections are drafted.

# Relationship to the Three Critical Rules

This skill enforces the three critical rules from the workflow specification:

1. **No final paper without final method explanation**: This skill verifies that `qx_final_method_explanation.md` exists before building the package. Without it, the package cannot be built, and the writer cannot proceed.

2. **No writer handoff without final result analysis**: This skill verifies that `qx_final_result_analysis.md` exists before building the package. Raw results are not enough — the programmer must have analyzed them.

3. **Writer reads only from the solution package**: This skill produces the single document (`qx_solution_package_for_writer.md`) that the paper writer should use as their primary source. The writer should not need to hunt through scattered results, figures, and method notes.

# Examples

## Example 1: Complete package for Q1

Input state:
- `methods/Q1/q1_final_method_explanation.md` exists (entropy-TOPSIS selected as final).
- `results/Q1/reports/q1_final_result_analysis.md` exists (final ranking, sensitivity).
- All figures exist on disk.
- Robustness report exists.

Output (summary):

```markdown
# Q1 Solution Package for Paper Writer

## 0. Quick Reference
- **Subquestion**: Evaluate and rank cities by resilience.
- **Final method**: Entropy-Weight TOPSIS.
- **Key result**: City A, B, C rank top 3 with stable ordering under ±10% weight perturbation.
- **Confidence**: High.

## 4. Figure and Table Assignments

| Figure File | Paper Section | Claim Supported | Draft Caption | Status |
|------------|---------------|-----------------|---------------|--------|
| `results/Q1/experiments/final/figures/q1_ranking_comparison.png` | Results Analysis | "Entropy-TOPSIS differentiates alternatives more clearly than equal-weight scoring." | "Figure X: Comparison of city rankings under equal-weight baseline and entropy-TOPSIS. The entropy-based method reveals clearer separation among middle-ranked cities." | exists |
```

## Example 2: Blocked — missing final method explanation

Input state:
- User says "build solution package for Q2".
- `results/Q2/reports/q2_final_result_analysis.md` exists.
- `methods/Q2/q2_final_method_explanation.md` does NOT exist.

Output:
```json
{
  "blocked_items": [
    "Q2 is NOT Ready for Writer. The final method explanation is missing at methods/Q2/q2_final_method_explanation.md."
  ],
  "what_exists": [
    "Final result analysis is available at results/Q2/reports/q2_final_result_analysis.md"
  ],
  "what_is_missing": [
    "methods/Q2/q2_final_method_explanation.md"
  ],
  "recommended_next_skill": "final-method-explainer",
  "recommended_next_action": "The modeler must confirm the final method for Q2, then run final-method-explainer to produce the method explanation document."
}
```

## Example 3: Package with robustness gap

Input state:
- All required files exist for Q3.
- Figures exist.
- No robustness report exists.

Output (excerpt from package):

```markdown
### 3C. For "Robustness and Sensitivity" Section

**What to write**:
⚠ NO ROBUSTNESS REPORT EXISTS for Q3.

**Writer guidance**: Do not fabricate robustness claims. Instead, in the paper section, state:
- "Constraint sensitivity analysis is planned but not yet completed."
- OR: "The optimization result is deterministic given fixed inputs; sensitivity to parameter uncertainty is addressed through scenario analysis in Section X."

**If the modeler or programmer can provide robustness results**, append them to this package before writing the final paper section.
```

## Example 4: Contradictory sources

Input state:
- Final method explanation says M2 was selected.
- Final result analysis describes outputs from M3.
- The method name mismatch exists.

Output:
```json
{
  "blocked_items": [
    "Source document contradiction: q1_final_method_explanation.md states the final method is 'M2: Entropy-TOPSIS', but q1_final_result_analysis.md describes results for 'M3: AHP-TOPSIS'. These must be reconciled before the package can be built."
  ],
  "recommended_next_skill": "workflow-orchestrator",
  "recommended_next_action": "The modeler and programmer must agree on which method is final and reconcile the two documents."
}
```

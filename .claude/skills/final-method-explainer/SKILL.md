---
name: final-method-explainer
description: Help the modeler write a comprehensive final method explanation document that justifies the selected method, documents eliminated alternatives, defines assumptions, symbols, objectives, and solution steps for paper writing.
license: MIT
---

# Purpose

Help the modeling team member write a definitive final method explanation for a subquestion after multi-round experimentation has converged on the best method.

This skill converts the candidate method pool, iteration logs, and experiment reports into a single authoritative document that explains:
- Which method was finally chosen and why.
- Which other candidate methods were eliminated and why.
- The full mathematical specification: assumptions, symbols, objective function, constraints, solution procedure.
- Evaluation metrics and how the method's output should be judged.
- Concrete paper-writing suggestions for the model construction section.

This skill does not select the final method (the modeler does), run experiments, write final paper sections directly, or generate code.

# When to use

Use this skill:

- After one or more rounds of experiments have been completed and the modeler has decided which method to use.
- After `result-report-generator` has produced experiment reports and the modeler has reviewed them.
- When the modeler needs to produce `methods/Qx/qx_final_method_explanation.md`.
- When the user says: "write the final method explanation for Q1", "document the final method", "explain why we chose this method", "prepare the method for paper writing".
- Before `solution-package-builder` or `paper-section-writer`.
- When the method iteration log shows convergence and the modeler confirms final selection.

# Preconditions

The following should already exist or be provided:

- The candidate method pool for this subquestion (`methods/Qx/qx_method_candidates.md` or equivalent).
- The method iteration log (`methods/Qx/qx_method_iteration_log.md` or equivalent), recording what changed across rounds.
- Experiment reports from at least one round (`results/Qx/experiments/roundN/qx_experiment_report_roundN.md`).
- The modeler's explicit confirmation of which method is the final choice (or clear evidence from iteration logs).
- The problem parse and classification artifacts (for subquestion goals, constraints, and required outputs).

If the candidate method pool does not exist, hand back to `method-selector`.

If no experiment reports exist, hand back to `result-report-generator`.

If the modeler has not confirmed the final method choice, ask for confirmation before producing the final explanation. Do not guess which method is "final" from experiment results alone.

# Inputs

Use or request:

- `methods/Qx/qx_method_candidates.md` — the original candidate method pool.
- `methods/Qx/qx_method_iteration_log.md` — the record of method changes across rounds.
- `results/Qx/experiments/roundN/qx_experiment_report_roundN.md` — experiment reports from all rounds.
- `workspace/problem/problem-parser/problem_parse.json` — for subquestion goals and constraints.
- `workspace/problem/problem-classifier/problem_classification.json` — for task type context.
- `workspace/problem/method-selector/method_plan.json` — for original method specifications.
- The modeler's explicit confirmation of the final method choice.
- Any additional notes the modeler provides about why they chose this method.

# Workflow

1. Confirm final method selection.
   - Read the iteration log to trace the decision path: what was tried, what failed, what was improved, what was dropped.
   - If the iteration log ends with a clear final choice, use it.
   - If the iteration log is ambiguous or the modeler has not confirmed, stop and ask for explicit confirmation.
   - Do not independently decide which method is "final" without modeler input or clear log evidence.

2. Document the selection process.
   - State the final selected method clearly.
   - List every candidate method that was considered and experimented.
   - For each eliminated method, state the specific evidence-based reason it was dropped (from experiment reports, not from speculation).
   - Explain why the final method was chosen: better metrics, better interpretability, better fit to contest constraints, robustness, simplicity, or combination thereof.
   - Be honest about trade-offs. If the final method has weaknesses, state them explicitly.

3. Define model assumptions.
   - List every modeling assumption the final method relies on.
   - Separate "necessary" assumptions (the model breaks without them) from "simplifying" assumptions (the model works but is more approximate without them).
   - For each assumption, state its possible impact on results.
   - Link assumptions to the problem context — why is this assumption reasonable for this specific contest problem?
   - Do not include generic assumptions that don't affect the model.

4. Define symbols and notation.
   - Create a complete symbol table covering all variables, parameters, sets, indices, functions, and outputs used in the final method.
   - Distinguish: decision variables, state variables, parameters, inputs, intermediate values, and outputs.
   - Include units where applicable.
   - Ensure notation is consistent with other subquestions (check the global symbol table if it exists).
   - Mark symbols that are shared across subquestions.

5. Write the mathematical model specification.
   - State the objective function (for optimization) or evaluation criterion (for evaluation) or prediction target (for prediction).
   - State all constraints in mathematical form.
   - State the solution procedure step by step.
   - For optimization: define decision variables, objective, constraints, and solver approach.
   - For evaluation: define indicator normalization, weighting method, aggregation formula, and ranking rule.
   - For prediction: define model form, training procedure, prediction formula, and error estimation.
   - For mechanism: define governing equations, parameters, boundary conditions, and solution method.
   - For classification/clustering: define features, model form, training/clustering procedure, and assignment rule.
   - For simulation: define stochastic elements, trial structure, output statistics, and seed management.
   - Every equation must have its variables defined. Every parameter must have a source or estimation method.

6. Define evaluation metrics.
   - State how this method's output quality is measured.
   - Define each metric formula.
   - State expected ranges or acceptable thresholds.
   - Link metrics to the subquestion's evaluation criteria from the problem parse.

7. Provide paper-writing guidance.
   - State which parts of this explanation should go into the "Model Construction" section of the paper.
   - State which formulas are most important to show.
   - State which assumptions should be highlighted.
   - State which eliminated methods should be mentioned (for demonstrating thoroughness).
   - Suggest a logical flow: assumptions → symbols → model formulation → solution → metrics.
   - Flag any parts of the explanation that are "for internal use only" and should not appear in the paper (e.g., implementation details, debugging notes).

8. Produce the final method explanation document.
   - Save as `methods/Qx/qx_final_method_explanation.md`.
   - Keep it structured, complete, and self-contained.
   - The document should be understandable without reading the iteration logs or experiment reports (though it should reference them).

9. Hand off to the next skill.
   - If the final result analysis also exists: hand off to `solution-package-builder`.
   - If the final result analysis does not yet exist: hand off to `result-report-generator` or `workflow-orchestrator`.

# Outputs

Produce exactly one document:

- `methods/Qx/qx_final_method_explanation.md`

# Output format

The document MUST contain all of the following sections:

```markdown
# Qx Final Method Explanation

## 1. Method Selection Summary

- **Final selected method**: [method name]
- **Selection date / round**: [when this was decided]
- **Modeler**: [name or role if applicable]

### Candidate Methods Considered

| Method | Status | Reason |
|--------|--------|--------|
| M1: [name] | Eliminated — Round N | [specific evidence-based reason] |
| M2: [name] | Selected — Final | [specific evidence-based reason] |
| M3: [name] | Eliminated — Round N | [specific evidence-based reason] |

### Selection Justification

- **Why this method was chosen**: [detailed explanation tied to experiment results]
- **Trade-offs accepted**: [weaknesses or compromises knowingly accepted]
- **Key differentiator**: [what made this method better than alternatives]

## 2. Model Assumptions

### Necessary Assumptions (model breaks without them)

| # | Assumption | Justification | Impact if Violated |
|---|-----------|---------------|-------------------|
| 1 | [assumption] | [why reasonable for this problem] | [what happens if not true] |

### Simplifying Assumptions (model works approximately without them)

| # | Assumption | Justification | Impact if Relaxed |
|---|-----------|---------------|------------------|
| 1 | [assumption] | [why acceptable] | [effect on results] |

## 3. Symbols and Notation

### Sets and Indices

| Symbol | Meaning | Domain / Range |
|--------|---------|---------------|
| [symbol] | [description] | [domain] |

### Parameters (given or estimated)

| Symbol | Meaning | Value / Source | Unit |
|--------|---------|---------------|------|
| [symbol] | [description] | [value or how obtained] | [unit] |

### Decision Variables (controlled by the model)

| Symbol | Meaning | Domain | Unit |
|--------|---------|--------|------|
| [symbol] | [description] | [domain] | [unit] |

### State Variables (computed from decisions and parameters)

| Symbol | Meaning | Computation | Unit |
|--------|---------|------------|------|
| [symbol] | [description] | [formula or source] | [unit] |

### Outputs (delivered to the paper / downstream)

| Symbol | Meaning | Format |
|--------|---------|--------|
| [symbol] | [description] | [form] |

## 4. Mathematical Model

### 4.1 Objective Function / Evaluation Criterion
[Mathematical formulation with explanation]

### 4.2 Constraints
[Each constraint in mathematical form with explanation]

### 4.3 Solution Procedure

**Step 1**: [what is done, with formula if applicable]
**Step 2**: [what is done, with formula if applicable]
...

### 4.4 Key Formulas Summary
[The most important formulas, collected for easy reference]

## 5. Evaluation Metrics

| Metric | Formula | Expected Range | Interpretation |
|--------|---------|---------------|----------------|
| [metric] | [formula] | [range] | [what good/bad values mean] |

## 6. Relationship to Other Subquestions

- **Inputs from other subquestions**: [what this method receives from Q1, Q2, etc.]
- **Outputs to other subquestions**: [what this method provides to Q2, Q3, etc.]
- **Shared symbols**: [symbols also used in other subquestions]
- **Consistency notes**: [any alignment needed]

## 7. Paper Writing Guidance

### Suggested Section Flow
1. [subsection name] — [what to write]
2. [subsection name] — [what to write]
...

### Formulas to Show in Paper
- [formula reference] — [why it's important, where to show it]

### Assumptions to Highlight
- [assumption] — [why it matters to the reader]

### Eliminated Methods to Mention
- [method] — [brief reason for elimination, to show methodology was thorough]

### Internal-Use Only (do NOT put in paper)
- [item] — [why it should stay internal]

## 8. References to Supporting Documents
- Candidate methods: `methods/Qx/qx_method_candidates.md`
- Iteration log: `methods/Qx/qx_method_iteration_log.md`
- Experiment reports: `results/Qx/experiments/roundN/`
- Final result analysis: `results/Qx/reports/qx_final_result_analysis.md` (when available)
```

# Rules

- Do not select the final method independently. The modeler must confirm.
- Every eliminated method must have a specific, evidence-based reason for elimination (cite experiment report findings).
- Every assumption must be linked to a modeling need — no filler assumptions.
- Every symbol must be defined before use in formulas.
- Notation must be checked against other subquestions for consistency.
- The solution procedure must be specific enough that a programmer could re-implement it.
- The paper writing guidance should be actionable, not vague ("show the formula" not "discuss the model").
- Do not fabricate experiment results, metrics, or selection reasons.
- Do not write the actual paper section — this is a method explanation for the modeler and paper writer, not the paper itself.
- Do not claim superiority that is not supported by experiment reports.
- Trade-offs and weaknesses must be stated honestly.
- If the method has known limitations, state them clearly.

# Verification

Before handing off, verify:

- The final method is explicitly named and confirmed.
- Every candidate method's fate (eliminated or selected) is documented with evidence-based reasons.
- Assumptions are categorized (necessary vs simplifying) and justified.
- The symbol table is complete and distinguishes variable types.
- Mathematical formulations are present and variables are defined.
- Solution procedure is step-by-step and implementable.
- Evaluation metrics are defined with formulas and interpretation.
- Paper-writing guidance is specific and actionable.
- Cross-subquestion consistency is checked (if other subquestions are available).
- The document is self-contained and understandable on its own.

# Failure modes

Stop and report a blocker if:

- The modeler has not confirmed the final method choice.
- The iteration log is empty or nonexistent — there is no decision path to document.
- No experiment reports exist — there is no evidence base for selection.
- The chosen method cannot be mathematically specified (missing variables, undefined parameters, unclear procedure).
- Cross-subquestion symbol conflicts exist and cannot be resolved without changes to other subquestions.
- The modeler asks for a method explanation that contradicts experiment evidence.

# Stop conditions

This skill must stop instead of guessing when:

- The final method choice is ambiguous or contested.
- Key parameters have no source and no reasonable default.
- The method cannot be written in mathematical form without inventing missing pieces.
- The experiment evidence contradicts the claimed selection reason.
- The modeler asks to fabricate favorable justifications or hide weaknesses.

When stopping, output:

- the blocker
- why it matters
- what is missing or contradictory
- what can still be partially documented
- recommended next action (usually: return to modeler for clarification)

# Handoff

After producing `methods/Qx/qx_final_method_explanation.md`, hand off to:

`solution-package-builder`
— if `results/Qx/reports/qx_final_result_analysis.md` also exists.

`workflow-orchestrator`
— if the final result analysis does not yet exist and the workflow needs to determine next steps.

`result-report-generator`
— if the final method is confirmed but the final result analysis has not been generated yet.

Do not hand off directly to `paper-section-writer`. The paper writer should receive the solution package, which bundles the final method explanation with the final result analysis.

# Examples

## Example 1: Final method explanation for evaluation problem (Q1)

Input state:
- Q1 candidate methods: equal-weight scoring (M1), entropy TOPSIS (M2), AHP-TOPSIS (M3).
- Iteration log shows M3 was dropped after round 1 (AHP subjective weights hard to justify).
- Round 2 compared M1 and M2. Modeler selected M2 as final, keeping M1 as baseline.
- Modeler confirms: "M2 entropy-TOPSIS is final."

Output:
```markdown
# Q1 Final Method Explanation

## 1. Method Selection Summary

### Candidate Methods Considered

| Method | Status | Reason |
|--------|--------|--------|
| M1: Equal-Weight Normalized Scoring | Retained as baseline | Provides transparent reference; no weight justification needed |
| M2: Entropy-Weight TOPSIS | **Selected — Final** | Objective data-driven weights; top-3 ranking stable under perturbation; better differentiation than M1 |
| M3: AHP-TOPSIS | Eliminated — Round 1 | Requires subjective pairwise comparison matrix; cannot be objectively justified without domain expert input |

### Selection Justification

- **Why M2 was chosen**: M2 (Entropy-TOPSIS) produces objective weights directly from indicator data dispersion, eliminating subjective bias. The resulting ranking differentiates alternatives more clearly than M1 while remaining stable under ±10% weight perturbation (see round 2 experiment report).
- **Trade-offs accepted**: Entropy weighting can overweight indicators with high variance that may not be practically important. This is noted as a limitation.
- **Key differentiator**: M2 is the only method that provides both objective weighting AND clear ranking scores, satisfying both contest scoring criteria and interpretability requirements.

## 3. Symbols and Notation

### Sets and Indices

| Symbol | Meaning | Domain / Range |
|--------|---------|---------------|
| i | City index | i = 1, 2, ..., m |
| j | Indicator index | j = 1, 2, ..., n |

### Parameters

| Symbol | Meaning | Value / Source | Unit |
|--------|---------|---------------|------|
| x_{ij} | Raw value of indicator j for city i | From cleaned indicator matrix | varies by indicator |

## 4. Mathematical Model

### 4.3 Solution Procedure

**Step 1 — Normalization**: Apply min-max normalization with direction handling.
[formula]

**Step 2 — Entropy weight calculation**: Compute entropy e_j for each indicator.
[formula]

**Step 3 — TOPSIS scoring**: Compute ideal best/worst and relative closeness.
[formula]

**Step 4 — Ranking**: Sort cities by C_i descending.
```

## Example 2: Blocked — modeler has not confirmed

Input state:
- Experiment reports exist for Q2, round 2.
- Iteration log shows three methods tested.
- No modeler confirmation of final choice.

Output:
```json
{
  "blocked_items": [
    "The modeler has not confirmed which method is final. The iteration log shows M2 and M4 both performed well in round 2."
  ],
  "safe_partial_work": [
    "Symbol table and assumption list can be drafted for both M2 and M4."
  ],
  "recommended_next_action": "Ask the modeler to explicitly confirm the final method choice for Q2 before producing the final method explanation."
}
```

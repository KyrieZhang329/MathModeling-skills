---
name: paper-section-writer
description: Write mathematical modeling paper sections based only on available problem analysis, model plans, results, figures, and robustness reports.
license: MIT
---

# Purpose

Write mathematical modeling paper sections from validated workflow artifacts.

This skill turns problem analysis, method plans, data reports, model results, robustness checks, and figure-table plans into structured paper section drafts. It must keep every claim traceable to an artifact and must not invent data, numerical results, figures, references, or unsupported conclusions.

This skill does not run models, generate new results, create unsupported figures, perform final QA, or approve final submission.

# When to use

Use this skill:

- After `figure-table-planner` has produced a validated figure and table plan.
- After model results and robustness evidence exist.
- When paper section drafts are needed.
- When existing section drafts need to be aligned with validated artifacts.
- Before `quality-assurance-auditor`.

# Preconditions

The following should already exist or be provided:

- A validated problem parse.
- A validated problem classification artifact.
- A validated method plan.
- Data audit report and cleaned data summary if data is used.
- Model result artifacts.
- Robustness or sensitivity report.
- Figure-table plan.
- Existing figures and tables, or clearly marked planned figures and tables.
- Contest formatting requirements if available.

If model results do not exist, hand back to `code-reviewer` or `model-code-generator`.

If the figure-table plan is missing, hand back to `figure-table-planner`.

If robustness evidence is required but missing, hand back to `robustness-checker`.

# Inputs

Use or request:

- `workspace/problem/problem_parse.json`, if available.
- `workspace/problem/problem_classification.json`, if available.
- `workspace/problem/method_plan.json`, if available.
- `workspace/results/data_report.md`, if available.
- Model result files under `workspace/results/`.
- Robustness report and sensitivity tables under `workspace/results/`.
- Figure-table plan.
- Existing figures under `workspace/figures/`.
- Existing paper section drafts under `workspace/paper_sections/`, if any.
- Required contest paper structure or formatting notes, if available.

# Workflow

1. Gather validated artifacts.
   - Identify the available problem parse, method plan, data report, model outputs, robustness report, and figure-table plan.
   - Separate generated artifacts from planned but not yet generated artifacts.
   - Do not use missing artifacts as if they exist.

2. Build a section map.
   - Map each subquestion to the sections that should answer it.
   - Map each model to its assumptions, variables, equations, results, figures, and tables.
   - Map each major claim to a supporting artifact.

3. Draft standard paper sections.
   - Write only the sections requested or appropriate for the current workflow stage.
   - Keep the writing concise, technical, and evidence-based.
   - Avoid filler prose.

4. Maintain artifact traceability.
   - Every numerical claim must point to a result artifact.
   - Every figure or table reference must appear in the figure-table plan or existing files.
   - Every robustness claim must point to a robustness artifact.
   - Every model description must align with the method plan.

5. Write formulas and symbols carefully.
   - Use mathematical notation only when variables are defined.
   - Keep notation consistent with the method plan.
   - Do not introduce new variables without defining them.
   - Distinguish decision variables, state variables, parameters, inputs, and outputs when relevant.

6. Handle missing evidence explicitly.
   - If a result, figure, table, or robustness check is missing, mark the section as incomplete.
   - Do not fill gaps with invented values or generic claims.
   - Recommend the upstream skill needed to complete the missing evidence.

7. Produce section drafts.
   - Save or recommend saving drafts under `workspace/paper_sections/`.
   - Use section-oriented file names.
   - Preserve a clear link between drafts and source artifacts.

8. Recommend final audit.
   - After section drafts are complete enough, hand off to `quality-assurance-auditor`.

# Outputs

Produce paper section drafts and a writing summary such as:

- `workspace/paper_sections/abstract.md`
- `workspace/paper_sections/problem_restatement.md`
- `workspace/paper_sections/problem_analysis.md`
- `workspace/paper_sections/assumptions.md`
- `workspace/paper_sections/symbols.md`
- `workspace/paper_sections/data_preprocessing.md`
- `workspace/paper_sections/model_q1.md`
- `workspace/paper_sections/model_q2.md`
- `workspace/paper_sections/model_q3.md`
- `workspace/paper_sections/robustness.md`
- `workspace/paper_sections/strengths_limitations.md`
- `workspace/paper_sections/conclusion.md`
- section-to-artifact mapping
- incomplete claims list
- recommended next skill

# Output format

Prefer this JSON-compatible summary:

```json
{
  "paper_writing_summary": {
    "status": "drafted_with_gaps",
    "drafted_sections": [
      "workspace/paper_sections/problem_restatement.md",
      "workspace/paper_sections/model_q1.md",
      "workspace/paper_sections/robustness.md"
    ],
    "incomplete_sections": [
      {
        "section": "conclusion",
        "reason": "Final QA has not confirmed whether all subquestions are fully answered."
      }
    ],
    "unsupported_claims": [
      {
        "claim": "The model is globally stable.",
        "reason": "Robustness checks only cover moderate weight perturbation."
      }
    ],
    "artifact_mapping": [
      {
        "section": "model_q1.md",
        "uses_artifacts": [
          "workspace/problem/method_plan.json",
          "workspace/results/q1_main_results.csv",
          "workspace/figures/q1_ranking_comparison.png"
        ]
      }
    ],
    "recommended_next_skill": "quality-assurance-auditor"
  }
}
```

If a JSON block is too rigid for the situation, use a concise Markdown report with the same fields.

# Paper section types

Use these section types when appropriate.

## Abstract

Purpose:

- Summarize the problem, methods, key results, robustness evidence, and final conclusions.

Requirements:

- Mention the main task.
- Mention the main methods actually used.
- Mention only numerical results that exist.
- Mention robustness only if robustness checks exist.
- Keep concise.

Do not:

- invent final scores, rankings, errors, or optimal values
- claim superiority without comparison evidence
- include unsupported broad claims

## Problem restatement

Purpose:

- Restate the problem in the team's own words while preserving the original meaning.

Requirements:

- Include background, main goal, subquestions, required outputs, and constraints.
- Avoid adding new requirements not present in the problem.

## Problem analysis

Purpose:

- Explain how the problem is decomposed and why the workflow order is reasonable.

Requirements:

- Map each subquestion to its task type.
- Explain dependencies between subquestions.
- Explain why later sections follow the chosen order.

## Assumptions

Purpose:

- State simplifying assumptions needed for modeling.

Requirements:

- Each assumption should be necessary, interpretable, and linked to a modeling need.
- Mention possible impact when an assumption is strong.

Do not:

- use generic assumptions that do not affect the model
- hide missing data as an assumption

## Symbols and definitions

Purpose:

- Define variables, parameters, sets, indices, functions, and outputs.

Requirements:

- Distinguish decision variables, state variables, parameters, inputs, and outputs.
- Include units where available.
- Keep notation consistent across sections.

## Data preprocessing

Purpose:

- Explain data sources, field meanings, cleaning operations, and readiness.

Requirements:

- Use the data report.
- Mention missing values, outliers, units, transformations, and remaining risks.
- Do not claim raw data was changed if cleaned copies were used.

## Model construction

Purpose:

- Present the baseline, main model, and optional improved model for each subquestion.

Requirements:

- Align with the method plan.
- Define objectives, constraints, variables, or equations where applicable.
- Explain why the model fits the task type and data.
- Include baseline before claiming improvement.

## Model solution

Purpose:

- Explain how the model was solved or computed.

Requirements:

- Link to generated and reviewed scripts.
- Mention solver, algorithm, procedure, or computation pipeline where relevant.
- Avoid unsupported implementation details.

## Results analysis

Purpose:

- Interpret model outputs.

Requirements:

- Use result tables and figures.
- Keep conclusions proportional to evidence.
- Separate result description from causal interpretation.

## Robustness and sensitivity analysis

Purpose:

- Show whether conclusions are stable.

Requirements:

- Use robustness-checker outputs.
- Separate stable and fragile conclusions.
- State conclusion boundaries.

## Strengths and limitations

Purpose:

- Explain what the model does well and where it may fail.

Requirements:

- Be specific.
- Link limitations to assumptions, data, method, or robustness findings.

## Conclusion and recommendations

Purpose:

- Answer the original subquestions and provide final recommendations.

Requirements:

- Every conclusion should map to a subquestion.
- Avoid conclusions not supported by results or QA.

## Appendix notes

Purpose:

- Describe code, extra tables, parameter settings, or supplementary derivations.

Requirements:

- Link to scripts and outputs.
- Do not use appendices to hide unsupported reasoning.

# Writing rules

- Write from validated artifacts, not memory or guesswork.
- Keep every major claim traceable.
- Use concise, technical language.
- Prefer clear structure over ornate language.
- Do not write numerical claims without result artifacts.
- Do not cite figures or tables that do not exist or are only planned unless explicitly marked as planned.
- Do not treat planned figures as generated evidence.
- Do not invent references.
- Do not invent experiments.
- Do not invent parameter values.
- Do not inflate conclusions beyond robustness evidence.
- Do not hide uncertainty.
- If evidence is missing, mark the section incomplete and name the missing artifact.
- Avoid filler phrases that do not advance the argument.

# Rules

- Do not run code.
- Do not clean data.
- Do not change models.
- Do not fabricate data, results, references, figures, or tables.
- Do not perform final QA.
- Do not approve final submission.
- Do not overwrite validated artifacts without permission.
- Do not introduce notation that conflicts with the method plan.
- Do not merge final paper unless QA passes.
- Keep section drafts modular and reviewable.
- Mark unsupported claims explicitly.

# Verification

Before handing off, verify:

- Every drafted section uses only available artifacts.
- Every numerical claim is backed by a result file.
- Every figure or table reference maps to the figure-table plan or existing file.
- Every robustness claim maps to a robustness artifact.
- Every subquestion has a corresponding draft section or is marked incomplete.
- Assumptions are connected to modeling needs.
- Variables and notation are defined before use.
- Limitations are not hidden.
- Unsupported claims are listed.
- The next skill is `quality-assurance-auditor`.

# Failure modes

Stop and report a blocker if:

- Model results are missing for a section that requires them.
- Figure-table plan is missing.
- Robustness results are missing for a robustness claim.
- A requested section would require invented numerical values.
- A claim cannot be traced to any artifact.
- The user asks for final paper assembly before QA.
- The user asks to hide uncertainty or unsupported limitations.

# Stop conditions

This skill must stop instead of guessing when:

- Drafting would require inventing data, results, figures, tables, references, parameters, or experiments.
- A section depends on a missing upstream artifact.
- The available artifacts contradict each other.
- A conclusion would exceed the evidence.
- A figure or table reference cannot be traced.
- Continuing would overwrite reviewed section drafts without permission.

When stopping, output:

- the blocker
- why it matters
- affected section
- missing artifact or evidence needed
- safe partial draft if any
- recommended next action

# Handoff

After producing paper section drafts, hand off to:

`quality-assurance-auditor`

The handoff should include:

- drafted section paths
- artifact mapping
- figure and table references
- unsupported or incomplete claims
- missing artifacts
- known writing risks
- sections requiring human review

If missing model results block writing, hand back to:

`code-reviewer` or `model-code-generator`

If missing robustness evidence blocks writing, hand back to:

`robustness-checker`

If missing visual planning blocks writing, hand back to:

`figure-table-planner`

Do not hand off directly to final assembly unless `quality-assurance-auditor` has passed.

# Examples

## Example 1: Draft a model section with traceable artifacts

Input state:

- Q1 method plan exists.
- Q1 result table exists.
- Figure-table plan maps Q1 ranking table to the Q1 result section.

Output:

```json
{
  "paper_writing_summary": {
    "status": "drafted",
    "drafted_sections": [
      "workspace/paper_sections/model_q1.md"
    ],
    "artifact_mapping": [
      {
        "section": "model_q1.md",
        "uses_artifacts": [
          "workspace/problem/method_plan.json",
          "workspace/results/q1_main_results.csv",
          "workspace/figures/q1_ranking_comparison.png"
        ]
      }
    ],
    "unsupported_claims": [],
    "recommended_next_skill": "quality-assurance-auditor"
  }
}
```

## Example 2: Block unsupported numerical conclusion

Input state:

- User asks to write “the proposed model improves accuracy by 20%.”
- No error comparison table exists.

Output:

```json
{
  "blocked_items": [
    "The requested numerical improvement claim has no supporting result artifact."
  ],
  "affected_section": "results_analysis",
  "missing_artifact": "baseline vs main model error comparison table",
  "recommended_next_skill": "robustness-checker",
  "recommended_next_action": "Generate or provide the comparison result before writing this claim."
}
```

## Example 3: Draft with incomplete conclusion

Input state:

- Sections for Q1 and Q2 exist.
- Q3 model result is missing.
- User asks for conclusion.

Output:

```json
{
  "paper_writing_summary": {
    "status": "partial_draft_only",
    "drafted_sections": [
      "workspace/paper_sections/conclusion_partial.md"
    ],
    "incomplete_sections": [
      {
        "section": "conclusion",
        "reason": "Q3 has no validated result, so the final conclusion cannot answer all subquestions."
      }
    ],
    "recommended_next_skill": "workflow-orchestrator"
  }
}
```

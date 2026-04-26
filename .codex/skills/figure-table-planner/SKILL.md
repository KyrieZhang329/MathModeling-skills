---
name: figure-table-planner
description: Plan figures and tables that support the modeling logic, results, and paper narrative.
license: MIT
---

# Purpose

Plan figures and tables for a mathematical modeling contest solution.

This skill determines which figures and tables are needed, what claim each artifact supports, which subquestion it belongs to, which data or result artifact it should use, and where it should appear in the paper. It ensures that visual materials serve the modeling logic rather than decoration.

This skill does not fabricate data, generate unsupported figures, write final paper sections, or perform final QA.

# When to use

Use this skill:

- After `robustness-checker` has produced usable robustness evidence.
- After model results, baseline comparisons, and major validation outputs exist.
- Before `paper-section-writer`.
- When the team needs a figure and table plan for the paper.
- When existing figures or tables need to be checked for narrative relevance.
- When the solution needs flowcharts, mechanism diagrams, result plots, comparison tables, or sensitivity tables.

# Preconditions

The following should already exist or be provided:

- A validated problem parse.
- A validated method plan.
- Data report and field mapping if data is used.
- Model result tables.
- Robustness or sensitivity results.
- Existing figure files if already generated.
- Expected paper structure or subquestion order if available.

If model results do not exist, hand back to `code-reviewer`, `model-code-generator`, or `robustness-checker`.

If robustness evidence is required but missing, hand back to `robustness-checker`.

# Inputs

Use or request:

- `workspace/problem/problem_parse.json`, if available.
- `workspace/problem/method_plan.json`, if available.
- `workspace/results/data_report.md`, if available.
- Model result files under `workspace/results/`.
- Robustness report and sensitivity tables under `workspace/results/`.
- Existing figures under `workspace/figures/`.
- Code review summary if it affects figure reliability.
- Expected paper section list if available.
- User preferences for visual style or contest formatting, if provided.

# Workflow

1. Identify paper claims and evidence needs.
   - Determine which claims require visual or tabular support.
   - Map claims to subquestions and model outputs.
   - Do not plan figures for unsupported claims.

2. Inventory available artifacts.
   - List existing result tables, robustness tables, and figure files.
   - Identify which artifacts are ready to use.
   - Identify which artifacts are missing but necessary.

3. Plan structural figures.
   - Plan overall workflow diagrams when the solution has multiple stages.
   - Plan model principle diagrams when equations or mechanisms are hard to explain in text.
   - Plan variable or data-flow diagrams when they clarify dependencies.

4. Plan result figures.
   - Choose plots that directly support model findings.
   - Prefer interpretable plots over decorative visuals.
   - Ensure each figure has a clear source artifact.

5. Plan tables.
   - Plan symbol tables, parameter tables, data summary tables, model comparison tables, result tables, and sensitivity tables as needed.
   - Avoid duplicating the same information in both a table and a figure unless each serves a distinct purpose.

6. Link figures and tables to paper sections.
   - Assign every figure or table to a section.
   - State what sentence or claim it supports.
   - Mark required captions and interpretation notes.

7. Check visual evidence quality.
   - Ensure every planned artifact uses available data, model results, or robustness results.
   - Mark artifacts that are missing source data.
   - Reject decorative visuals that do not support reasoning.

8. Produce a figure-table plan.
   - Keep the plan concise and traceable.
   - Recommend `paper-section-writer` only after the visual plan is coherent.

# Outputs

Produce a figure and table plan containing:

- `figure_table_summary`
- `planned_figures`
- `planned_tables`
- `existing_artifacts`
- `missing_artifacts`
- `section_mapping`
- `caption_requirements`
- `visual_risks`
- `recommended_next_skill`

# Output format

Prefer this JSON-compatible structure:

```json
{
  "figure_table_summary": {
    "status": "ready_with_missing_optional_items",
    "main_visual_strategy": "Use workflow figures for modeling logic, result plots for conclusions, and sensitivity tables for robustness evidence.",
    "recommended_next_skill": "paper-section-writer"
  },
  "planned_figures": [
    {
      "id": "Fig.1",
      "type": "workflow",
      "title": "Overall modeling workflow",
      "subquestion": "all",
      "paper_section": "Problem analysis or model overview",
      "source_artifacts": [
        "workspace/problem/problem_parse.json",
        "workspace/problem/method_plan.json"
      ],
      "supports_claim": "The solution follows a staged workflow from data processing to model validation.",
      "status": "planned",
      "caption_requirement": "Explain each stage and its output artifact."
    }
  ],
  "planned_tables": [
    {
      "id": "Table 1",
      "type": "symbol_table",
      "title": "Main symbols and definitions",
      "subquestion": "all",
      "paper_section": "Symbols and assumptions",
      "source_artifacts": [
        "workspace/problem/method_plan.json"
      ],
      "supports_claim": "Variables and parameters are consistently defined before modeling.",
      "status": "planned",
      "caption_requirement": "Include units and roles where applicable."
    }
  ],
  "existing_artifacts": [
    "workspace/results/q1_main_results.csv"
  ],
  "missing_artifacts": [
    {
      "artifact": "workspace/figures/q1_weight_sensitivity.png",
      "reason": "Needed to support ranking stability claim."
    }
  ],
  "visual_risks": [
    "Do not include figures without source artifacts."
  ]
}
```

If a JSON block is too rigid for the situation, use a concise Markdown report with the same fields.

# Figure and table types

Use these artifact types consistently.

## Workflow figure

Use for:

- full modeling pipeline
- staged subquestion dependency
- data processing to result flow
- algorithm or decision process overview

Requirements:

- each arrow should represent a real transformation or handoff
- each block should correspond to a method, data step, or output
- avoid decorative flowcharts with vague labels

## Principle figure

Use for:

- mechanism explanation
- geometric relation
- variable interaction
- state transition
- model structure

Requirements:

- must clarify a formula, assumption, or system relation
- must use defined variables where possible
- must not replace a missing derivation

## Result figure

Use for:

- trends
- rankings
- prediction vs actual
- residuals
- spatial patterns
- comparison between baseline and main model

Requirements:

- must be generated from result artifacts
- must support a specific conclusion
- axes, units, labels, and legends must be interpretable

## Robustness figure

Use for:

- sensitivity curves
- perturbation effects
- ranking stability
- error distribution
- seed stability
- constraint sensitivity

Requirements:

- must come from completed robustness checks
- planned checks must not be shown as completed results

## Data summary table

Use for:

- data fields
- units
- missing values
- descriptive statistics
- preprocessing actions

Requirements:

- must match `data-auditor-cleaner` outputs
- must distinguish raw fields from derived features

## Model result table

Use for:

- scores
- rankings
- predictions
- optimal decisions
- parameter estimates
- objective values

Requirements:

- must link to model output files
- must include units where applicable
- must not include invented numerical results

## Comparison table

Use for:

- baseline vs main model
- method alternatives
- scenario comparisons
- before/after comparison

Requirements:

- must state comparison criteria
- must not exaggerate improvement

## Sensitivity table

Use for:

- parameter perturbation
- weight perturbation
- constraint changes
- scenario results

Requirements:

- must match robustness-checker outputs
- must indicate perturbation size and conclusion effect

## Symbol or parameter table

Use for:

- variables
- parameters
- units
- roles
- sources

Requirements:

- must distinguish decision variables, state variables, parameters, and outputs when relevant
- must be consistent with method plan and paper notation

# Planning rules

- Every planned figure or table must support a specific claim.
- Every planned figure or table must map to at least one source artifact.
- Every planned figure or table must map to a paper section.
- Prefer fewer high-value visuals over many decorative visuals.
- Do not duplicate the same evidence unnecessarily.
- Use workflow figures to explain logic, not results.
- Use result figures to support conclusions, not to decorate.
- Use tables when exact values matter.
- Use plots when patterns, trends, or comparisons matter.
- Use robustness visuals only when checks are completed or clearly marked as planned.
- Captions must explain what the reader should learn from the artifact.

# Rules

- Do not fabricate figure data.
- Do not fabricate table values.
- Do not plan visuals for unsupported claims.
- Do not treat decorative diagrams as evidence.
- Do not write full paper sections.
- Do not perform final QA.
- Do not change model results.
- Do not hide missing artifacts.
- Do not mark planned artifacts as generated.
- Do not include figures that cannot be traced to data, model outputs, or robustness outputs.
- Mark visual risks explicitly.

# Verification

Before handing off, verify:

- Every planned figure has a type, source artifact, target section, and supported claim.
- Every planned table has a type, source artifact, target section, and supported claim.
- Existing artifacts and missing artifacts are separated.
- No figure or table relies on fabricated data.
- Robustness visuals correspond to completed or explicitly planned checks.
- Important model claims have at least one supporting figure or table, or a stated reason why text is sufficient.
- Captions or caption requirements are specified.
- The next skill is `paper-section-writer`.

# Failure modes

Stop and report a blocker if:

- Model results are missing.
- Robustness results are missing but required for key claims.
- A requested figure has no source artifact.
- A requested table would require invented values.
- The paper structure is too unclear to map visuals to sections.
- Existing figures contradict model results or cannot be traced.

# Stop conditions

This skill must stop instead of guessing when:

- A planned artifact requires unavailable numerical results.
- A figure or table would imply a claim not supported by existing outputs.
- The source data for a visual cannot be identified.
- The user asks to create evidence for a conclusion that has not been validated.
- Continuing would require inventing data, captions, values, or experimental findings.

When stopping, output:

- the blocker
- why it matters
- affected figure or table
- missing artifact or source needed
- safe partial plan if any
- recommended next action

# Handoff

After producing a validated figure-table plan, hand off to:

`paper-section-writer`

The handoff should include:

- figure-table plan
- planned figure list
- planned table list
- source artifacts
- section mapping
- caption requirements
- visual risks
- missing artifacts
- supported and fragile conclusions from robustness checks

If missing visuals require additional robustness results, hand back to:

`robustness-checker`

If missing visuals require model outputs, hand back to:

`code-reviewer` or `model-code-generator`

Do not hand off directly to `quality-assurance-auditor` unless paper sections already exist.

# Examples

## Example 1: Evaluation model visual plan

Input state:

- Q1 uses entropy-weight TOPSIS.
- Ranking results exist.
- Weight sensitivity results exist.

Output:

```json
{
  "planned_figures": [
    {
      "id": "Fig.1",
      "type": "workflow",
      "title": "Evaluation model workflow",
      "subquestion": "Q1",
      "paper_section": "Q1 model construction",
      "source_artifacts": [
        "workspace/problem/method_plan.json"
      ],
      "supports_claim": "The evaluation process follows indicator normalization, weighting, scoring, and ranking.",
      "status": "planned"
    },
    {
      "id": "Fig.2",
      "type": "robustness_figure",
      "title": "Ranking stability under weight perturbation",
      "subquestion": "Q1",
      "paper_section": "Robustness analysis",
      "source_artifacts": [
        "workspace/results/q1_weight_sensitivity.csv"
      ],
      "supports_claim": "Top-ranked alternatives remain stable under moderate weight changes.",
      "status": "ready_to_generate"
    }
  ],
  "planned_tables": [
    {
      "id": "Table 1",
      "type": "model_result_table",
      "title": "TOPSIS scores and rankings",
      "subquestion": "Q1",
      "paper_section": "Q1 results",
      "source_artifacts": [
        "workspace/results/q1_main_results.csv"
      ],
      "supports_claim": "The model produces comparable scores and rankings."
    }
  ],
  "recommended_next_skill": "paper-section-writer"
}
```

## Example 2: Prediction model visual plan

Input state:

- Q2 prediction results exist.
- Error metrics exist.
- Residual plot does not exist.

Output:

```json
{
  "planned_figures": [
    {
      "id": "Fig.3",
      "type": "result_figure",
      "title": "Actual vs predicted values",
      "subquestion": "Q2",
      "paper_section": "Q2 results",
      "source_artifacts": [
        "workspace/results/q2_predictions.csv"
      ],
      "supports_claim": "The prediction model captures the main observed trend.",
      "status": "ready_to_generate"
    }
  ],
  "planned_tables": [
    {
      "id": "Table 2",
      "type": "comparison_table",
      "title": "Prediction error comparison",
      "subquestion": "Q2",
      "paper_section": "Q2 validation",
      "source_artifacts": [
        "workspace/results/q2_error_metrics.csv"
      ],
      "supports_claim": "The main model improves short-term prediction error over the baseline."
    }
  ],
  "missing_artifacts": [
    {
      "artifact": "workspace/figures/q2_residual_plot.png",
      "reason": "Useful for error behavior discussion but not yet generated."
    }
  ],
  "recommended_next_skill": "paper-section-writer"
}
```

## Example 3: Block unsupported figure

Input state:

- User requests a chart showing that the model is always stable.
- Robustness results show only limited weight perturbation.

Output:

```json
{
  "blocked_items": [
    "The requested figure would overstate robustness beyond the completed checks."
  ],
  "visual_risks": [
    "Existing robustness results only support stability under limited weight perturbation, not universal stability."
  ],
  "recommended_next_action": "Either narrow the figure claim or return to robustness-checker for broader tests."
}
```

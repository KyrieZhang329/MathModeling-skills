---
name: result-report-generator
description: Generate multi-method experiment reports and final result analysis from model experiment outputs, comparing methods and providing actionable feedback for modelers and paper writers.
license: MIT
---

# Purpose

Generate structured experiment reports and final result analysis for each subquestion based on actual model experiment outputs, figures, tables, and metrics.

This skill converts raw experimental outputs into readable, comparative reports that:
- Explain what each method produced, numerically and visually.
- Compare multiple candidate methods head-to-head on defined metrics.
- Judge whether results are good enough to proceed or need method revision.
- Tell the modeler what to fix or reconsider.
- Tell the paper writer what results are usable, what claims are supported, and what visual evidence exists.

This skill does not run experiments, generate figures from scratch, select model methods, write final paper sections, or perform QA.

# When to use

Use this skill:

- After one or more candidate methods have been run and produced outputs under `results/Qx/experiments/roundN/`.
- When the team needs a comparative experiment report to decide which method to keep, drop, or revise.
- When the modeler asks "how did my methods perform?" and needs a structured answer.
- When the programmer has run experiments and needs to produce a structured handoff report for the modeler.
- When results are ready for the paper writer and need to be packaged into a readable narrative.
- When the user says: "generate experiment report for Q1", "compare methods for Q2", "write result analysis for Q3", "is the result good enough for the paper?", or "should I go back and revise the model?".

# Preconditions

The following should already exist or be provided:

- A validated method plan with candidate methods defined for the subquestion.
- Experimental outputs under `results/Qx/experiments/roundN/` including at minimum:
  - Method output files (scores, predictions, plans, etc.).
  - Figures saved in the `figures/` subdirectory.
  - Tables saved in the `tables/` subdirectory.
  - Metrics saved in the `metrics/` subdirectory (if available).
  - `run_summary.json` recording what was run (if available).
- Cleaned data paths and field mappings.
- Expected output definitions from the method plan.

If experimental outputs do not exist, hand back to the appropriate code generator or `workflow-orchestrator`.

If the method plan is missing, hand back to `method-selector`.

# Inputs

Use or request:

- `results/Qx/experiments/roundN/run_summary.json` or equivalent execution record.
- Method output files (CSV, JSON, MAT, etc.) under `results/Qx/experiments/roundN/`.
- Figures under `results/Qx/experiments/roundN/figures/`.
- Tables under `results/Qx/experiments/roundN/tables/`.
- Metrics under `results/Qx/experiments/roundN/metrics/`.
- Logs under `results/Qx/experiments/roundN/logs/` if available.
- The method plan for this subquestion (candidate methods, expected outputs, evaluation criteria).
- Cleaning reports or data field mappings if needed for interpreting results.
- Any user notes about method priorities, contest scoring, or expectations.

# Workflow

1. Inventory available outputs.
   - List every method that produced results in this round.
   - List every figure, table, metrics file, and log file.
   - Match each output to a candidate method from the method plan.
   - Mark missing expected outputs explicitly.

2. Read and interpret each method's results.
   - Extract key numerical outputs: scores, rankings, predictions, decisions, error values, objective values.
   - Read every figure: explain what it shows, whether the pattern is plausible, whether it reveals a problem.
   - Read every table: extract the main takeaway, check for anomalies.
   - Read metrics: compare against baseline, check against expected ranges.
   - Do not just list file names. Every artifact must be explained.

3. Compare methods.
   - Compare all methods in this round on the evaluation criteria defined in the method plan.
   - Use a comparison table or equivalent structured comparison.
   - State which method performs best on each criterion.
   - State trade-offs between methods (speed vs accuracy, simplicity vs precision, interpretability vs fit).
   - Do not hide poor performance.

4. Judge result quality and assign archival labels.
   - For each method, state whether results are: `[CHOSEN]` (good enough — keep in main tree), `[BACKUP]` (keep but secondary), or `[REJECTED]` (drop and archive).
   - Give a concrete reason for each judgment tied to metrics, visual evidence, or method-plan requirements.
   - If a method's output is clearly wrong (infeasible, contradictory, nonsensical), flag it `[REJECTED]` with explicit reason.
   - **For every `[REJECTED]` method**, move its outputs from the main tree to the archive:
     - Move `code/Qx/<method>_main.py` (or `.m`) → `workspace/archived/<Qx>/<method>_REJECTED_roundN/`.
     - Move its outputs in `results/Qx/experiments/roundN/figures/<method>_*` and `tables/<method>_*` → same archive directory.
     - Leave a one-line breadcrumb in `methods/Qx/qx_method_iteration_log.md` (e.g., "Round 1: M3 [REJECTED] — constraint infeasibility, archived to workspace/archived/Q3/m3_REJECTED_round1/").
   - The main tree under `code/Qx/` and `results/Qx/experiments/` should contain ONLY `[CHOSEN]` and `[BACKUP]` methods after this step. `[REJECTED]` outputs in the main tree are a workflow defect — they drift into the paper by accident.

5. Decide on iteration.
   - State unambiguously whether this subquestion should:
     - Proceed to final method selection (results are good enough).
     - Return to the modeler for method revision (results are poor or methods need redesign).
     - Run another round of experiments (more tuning or alternative methods needed).
   - If returning to modeler, state specifically what about the method is failing and what the modeler should reconsider.

6. Identify paper-ready materials.
   - List which figures and tables can go into the paper directly.
   - List which figures and tables need improvement before paper use.
   - List which numerical claims are supported by the current results.
   - List which claims are not yet supported and what additional evidence is needed.

7. Produce the experiment report (for round N).
   - Save as `results/Qx/experiments/roundN/qx_experiment_report_roundN.md`.
   - If this is the final round and results are locked in, also produce the final result analysis.

8. Produce the final result analysis (when all rounds are complete and final method is selected).
   - Save as `results/Qx/reports/qx_final_result_analysis.md`.
   - This is the definitive result document for the paper writer.
   - Only produce this when the modeler has confirmed the final method.

9. Hand off to the appropriate next skill.
   - If results need method revision: hand off to `final-method-explainer` or `method-selector`.
   - If results are final: hand off to `final-method-explainer` or `solution-package-builder`.
   - If a report is intermediate (round N, more rounds expected): hand off to `workflow-orchestrator`.

# Outputs

Produce the following artifacts as appropriate for the current stage:

- `results/Qx/experiments/roundN/qx_experiment_report_roundN.md` — Intermediate report comparing all methods run in this round.
- `results/Qx/reports/qx_final_result_analysis.md` — Final comprehensive result analysis (only when method is locked).

Each report type has different content requirements (see below).

# Output format

## Round experiment report (`qx_experiment_report_roundN.md`)

Must contain:

```markdown
# Qx Experiment Report — Round N

## 1. Execution Summary
- Methods run: [list]
- Execution date/time
- Input data used
- run_summary.json contents summary

## 2. Per-Method Results

### Method M1: [name]  [CHOSEN | BACKUP | REJECTED]
- **Key outputs**: [numerical summary]
- **Figures generated**: [list with one-sentence interpretation each]
- **Tables generated**: [list with one-sentence interpretation each]
- **Metrics**: [values against expected ranges]
- **Verdict**: [CHOSEN — keep in main tree / BACKUP — secondary / REJECTED — archived]
- **Reason**: [concrete evidence-based reason]
- **Archive action** (only if REJECTED): "Moved code/Qx/m1_main.py and results/Qx/experiments/roundN/figures/m1_*.png to workspace/archived/Qx/m1_REJECTED_roundN/"

### Method M2: [name]
...

## 3. Cross-Method Comparison

| Criterion | M1 | M2 | M3 | Best |
|-----------|---|---|---|------|
| [criterion] | [value] | [value] | [value] | [best] |

## 4. Overall Assessment
- **Round verdict**: [proceed to final / return to modeler / run another round]
- **If returning to modeler**: [specific issues and suggestions]
- **If proceeding**: [which method is recommended and why]

## 5. Paper-Ready Materials
- **Can use now**: [figures/tables/claims ready for paper]
- **Needs improvement**: [figures/tables/claims needing revision]
- **Not yet supported**: [claims lacking evidence]

## 6. Recommended Next Step
- [next skill or action]
- [what the next person should do]
```

## Final result analysis (`qx_final_result_analysis.md`)

Must contain:

```markdown
# Qx Final Result Analysis

## 1. Method Selection Summary
- Final selected method: [name]
- Eliminated methods and reasons for elimination
- Selection criteria and how the final method performed on each

## 2. Final Results
- **Key numerical outputs**: [comprehensive results]
- **Final figures** (with path and interpretation):
  - [figure path]: [what it shows, what claim it supports]
- **Final tables** (with path and interpretation):
  - [table path]: [what it shows, what claim it supports]

## 3. Result Quality Assessment
- **Confidence level**: [high / medium / needs caution]
- **Limitations**: [what this result cannot claim]
- **Stability**: [known from robustness checks or flagged as unchecked]

## 4. Paper Writing Support
- **Supported claims** (ready for paper):
  - [claim] — supported by [evidence]
- **Claims to avoid or qualify**:
  - [claim] — not fully supported because [reason]
- **Figure placement suggestions**:
  - [figure] → recommended section [section name]
- **Table placement suggestions**:
  - [table] → recommended section [section name]

## 5. Handoff
- **Ready for**: [final-method-explainer, solution-package-builder, or paper-section-writer]
- **Pending items**: [anything still missing]
```

# Rules

- Do not list file names without explaining what they contain.
- Every figure must be interpreted: what pattern, trend, or anomaly does it show?
- Every table must be interpreted: what is the key takeaway?
- Every method must receive a clear verdict (`[CHOSEN] / [BACKUP] / [REJECTED]`) with an evidence-based reason.
- **`[REJECTED]` methods must be archived**, not just labeled. Move their code and outputs to `workspace/archived/<Qx>/<method>_REJECTED_roundN/`. Leaving them in the main tree is a workflow defect.
- Comparison must be on explicit criteria, not vague impressions.
- Poor results must be reported honestly. Do not sugarcoat.
- If results are insufficient, state exactly what the modeler should reconsider.
- Do not fabricate metrics, figures, tables, or interpretation details.
- Do not claim results are "good" without defining what "good" means for this subquestion.
- Do not write full paper sections — this is a report, not a paper draft.
- Do not change the method plan or rerun experiments.
- Mark unsupported claims explicitly.
- Separate the experiment report (round-level, comparative) from the final result analysis (definitive, writer-facing).

# Verification

Before handing off, verify:

- Every method executed in this round has an interpreted summary.
- Every figure and table is interpreted, not just listed.
- A comparison table or structured comparison exists.
- Verdicts are supported by specific evidence.
- Paper-ready materials are distinguished from materials needing work.
- The recommended next action is clear and actionable.
- No fabricated numbers, figures, or interpretations exist.
- If returning to modeler, the feedback is specific enough to act on.

# Failure modes

Stop and report a blocker if:

- No experimental outputs exist for the requested subquestion or round.
- The method plan is missing, so there is no basis for judging results.
- All methods failed to produce meaningful outputs (all errors, no results).
- Outputs are present but uninterpretable without additional context that is unavailable.
- The user asks for a final result analysis but no method has been finalized by the modeler.
- The user asks to fabricate favorable interpretations for poor results.

# Stop conditions

This skill must stop instead of guessing when:

- Required output files cannot be read or are corrupted.
- The mapping between outputs and methods is ambiguous and cannot be resolved.
- Important metrics are missing and cannot be inferred from available outputs.
- Figures cannot be interpreted without the code that generated them and the code is unavailable.
- Interpretation would require inventing data, metrics, or causal claims.
- The user asks for a "final" report but the modeler has not confirmed the final method.

When stopping, output:

- the blocker
- why it matters
- affected artifacts
- what is still interpretable
- missing information or decision needed
- recommended next action

# Handoff

After producing an experiment report (round N, not final):

`workflow-orchestrator`
— provides the round report and indicates whether more rounds are needed.

After producing a final result analysis (final method locked):

`solution-package-builder` or `final-method-explainer`
— provides the final result analysis, figure paths, table paths, and supported claims.

If the report indicates method revision is needed:

`workflow-orchestrator`
— provides the specific issues found, affected methods, and suggestions for the modeler.

If the report indicates more experiments are needed (same methods, different parameters):

`workflow-orchestrator`
— provides the parameter adjustments or changes needed.

Do not hand off directly to `paper-section-writer` unless `solution-package-builder` has packaged the results first (or the workflow explicitly allows a direct handoff when all other prerequisites are met).

# Examples

## Example 1: Round 1 experiment report — proceed

Input state:
- Q1 has 3 methods run in round 1: equal-weight baseline (M1), entropy TOPSIS (M2), AHP-TOPSIS (M3).
- Figures: ranking comparison chart, weight bar chart.
- Metrics: ranking stability scores.

Output (summary):

```markdown
# Q1 Experiment Report — Round 1

## 3. Cross-Method Comparison

| Criterion | M1 Equal-Weight | M2 Entropy-TOPSIS | M3 AHP-TOPSIS | Best |
|-----------|----------------|-------------------|---------------|------|
| Ranking stability | High | High | Medium | M1, M2 |
| Weight interpretability | None | Objective (entropy) | Subjective (AHP) | M2 |
| Implementation simplicity | Very simple | Moderate | Complex (pairwise comparisons) | M1 |
| Differentiation power | Low | High | High | M2, M3 |

## 4. Overall Assessment
- **Round verdict**: proceed to final — M2 is recommended as the main method.
- **Reason**: M2 (Entropy-TOPSIS) provides objective data-driven weights with high stability and good differentiation. M3 (AHP-TOPSIS) requires subjective pairwise comparisons that are hard to justify without domain expert input. M1 is retained as a baseline reference for the paper.

## 6. Recommended Next Step
- hand off to final-method-explainer to lock in M2 as the final method.
```

## Example 2: Round 1 experiment report — return to modeler

Input state:
- Q3 has 2 methods run: greedy allocation (M1) and linear programming (M2).
- M1 produces feasible plans but with very high cost.
- M2 produces infeasible output because constraints were incorrectly specified.

Output (summary):

```markdown
# Q3 Experiment Report — Round 1

## 4. Overall Assessment
- **Round verdict**: return to modeler.
- **Issues to fix**:
  - M2 (linear programming): The output violates resource constraints. The modeler should re-check constraint formulation — specifically, whether capacity limit `C_max = 100` is correct and whether demand requirements are properly bounded.
  - M1 (greedy): Works correctly but cost is 45% higher than a theoretical lower bound. The modeler should decide whether a 45% cost gap is acceptable or whether a better method is needed.
  - Suggestion: if constraints are fixed in M2, run a new round with corrected M2. If not, consider whether M1 is "good enough" for contest purposes.

## 6. Recommended Next Step
- hand off to method-selector or the modeler to fix constraint specification for M2.
```

## Example 3: Final result analysis

Input state:
- Final method (entropy-TOPSIS) locked by modeler for Q1.
- Final round results complete.
- Robustness check results available.

Output (summary):

```markdown
# Q1 Final Result Analysis

## 4. Paper Writing Support

**Supported claims** (ready for paper):
- "Entropy-weighted TOPSIS produces a stable ranking with the top 3 cities remaining consistent under ±10% weight perturbation." — supported by q1_weight_sensitivity.csv and q1_ranking_comparison.png.
- "The model differentiates cities more clearly than equal-weight scoring." — supported by baseline comparison table.

**Claims to avoid or qualify**:
- Do not claim "the ranking is universally optimal." — supported only under the given indicator set and weight scheme.
```


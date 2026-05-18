# MathModeling-skills

[English](./README.md) | [简体中文](./README-zh.md)

A set of Claude Code / Codex Skills for mathematical modeling contests, breaking the contest workflow into small stages that can be completed step by step.

## Project Motivation

The real problems in mathematical modeling contests often do not come from “not knowing enough models.” They usually come from:

- misreading the problem and solving something the problem never asked for
- jumping to a complex model before testing a baseline
- writing code that runs but does not match the method described in the paper
- putting numbers in the paper that cannot be found in any script output
- claiming “our model is better” without a baseline or sensitivity analysis
- making good-looking figures that do not clearly support any conclusion

`MathModeling-skills` is designed around these issues. It splits the contest process into a group of small skills. Each skill has one narrow job, and the agent is expected to finish the current stage before moving to the next one.

## What It Does

- Splits the contest process into **26 single-purpose skills**, each with one narrow job.
- Replaces "advance by stage" with **6 explicit gates (G1–G6)**, each with `enter_condition / pass_criteria / fail_fallback`; G2 (method→code) and G4 (results→paper) are the load-bearing convergence points.
- Replaces single-point QA with a **3-auditor independent layer** (`consistency-auditor` cross-media, `completeness-auditor` on-disk audit-file check, `quality-assurance-auditor` workflow integrity) — any one failing blocks final assembly, so no skill can self-declare "done".
- Locks results→paper numbers with an immutable **`frozen_numbers.json`** snapshot, preventing silent drift after a bug fix.
- Keeps intermediate artifacts (parsed problems, method plans, PoC scripts, data reports, results, figures, frozen snapshots, paper drafts, audit reports) so every important number is traceable to its source.
- Uses the same skill set for Claude Code (`.claude/skills/`) and Codex (`.codex/skills/`).

## Project Boundaries

- It does not write a complete contest paper in one click.
- It does not invent data, results, references, or figures when evidence is missing. Missing evidence is reported as a blocker.
- It does not write numerical conclusions before results exist.
- It does not claim a model is better without a baseline and robustness or sensitivity checks.
- It does not modify your raw data. Cleaning should happen on copied data.
- It does not replace modeling judgment. The final choice of method still belongs to the user.

## Skills (26)

| Skill                       | Purpose                                                      |
| --------------------------- | ------------------------------------------------------------ |
| `workflow-orchestrator`     | Orchestrates per-question state via Gates G1–G6; pings the environment (Python / MATLAB / git) at session start. |
| `problem-parser`            | Reads the problem and extracts goals, objects, constraints, data, outputs, and subquestions. |
| `problem-classifier`        | Classifies each subquestion into a task type (evaluation, prediction, optimization, …). |
| `related-paper-analyzer`    | Collects and analyzes relevant papers before final method selection; no fabricated citations. |
| `method-selector`           | Proposes 2–4 candidates per subquestion; **each candidate must include a ≤30-line PoC + feasibility number** (Gate G2). Rejected candidates auto-archived. |
| `symbol-table-builder`      | Builds a unified symbol table across all subquestions. |
| `model-assumptions-builder` | Extracts and organizes global model assumptions (necessary vs simplifying). |
| `data-auditor-cleaner`      | Audits data, reports issues, produces cleaned data (copy-only, never touches raw). |
| `model-code-analyzer`       | Plans `experiments/roundN/` structure + `run_summary.json` schema before code generation. |
| `python-model-code-generator` | Generates Python modeling code when `implementation.target = python`. |
| `matlab-model-code-generator` | Generates MATLAB / 北太天元 compatible code when `implementation.target = matlab`. |
| `code-reviewer`             | Router: inspects script language and hands off to the right reviewer. |
| `python-code-reviewer`      | Reviews Python code; **must write `code/Qx/reviews/qx_python_review.md` with ≥ 5 explicit pass items** (Gate G3). |
| `matlab-code-reviewer`      | Reviews MATLAB code; **must write the equivalent review file with ≥ 5 explicit pass items** (Gate G3). |
| `result-report-generator`   | Multi-method experiment report + final result analysis; failed methods labeled `[REJECTED]` and moved to `workspace/archived/`. |
| `robustness-checker`        | Runs sensitivity, error, and baseline comparison checks. |
| `final-method-explainer`    | Helps the modeler document the final selected method. |
| `figure-table-planner`      | Plans the figures and tables that are actually needed. |
| `math-figure-generator`     | Publication-quality matplotlib figures; **mandatory `render_check_and_log()`** for bbox overlap, out-of-canvas text, fonts < 6.5pt. Type 3 promotion blocked on failure. |
| `solution-package-builder`  | Builds the writer-facing package + **emits the immutable `frozen_numbers.json` snapshot** (Gate G4). |
| `paper-section-writer`      | Drafts paper sections; **enforces section word-count floors + ≥ 3 discussion dimensions per numerical result** (sensitivity / physical / baseline / cross-Qx / uncertainty) (Gate G5). |
| `paper-polisher`            | Polishes syntax / tense / hedging / overclaim / within-document formula consistency. |
| `reference-manager`         | Generates BibTeX; detects fabricated citations. |
| **`consistency-auditor`**   | **[Audit layer 1/3] Cross-media check** for numbers / file names / symbols / parameters across tex ↔ code ↔ frozen_numbers.json ↔ symbol_table. Any divergence fails. |
| **`completeness-auditor`**  | **[Audit layer 2/3] Disk-artifact check**: every `*_review.md` / `*_audit.md` that should exist must exist with ≥ 5 explicit pass items. Verbal "passed" doesn't count. |
| `quality-assurance-auditor` | [Audit layer 3/3] Workflow completeness + three critical rules + anti-fabrication. Gate G6 requires all three audits PASS before final assembly. |

## Standard Workflow (with Gates G1–G6)

```text
workflow-orchestrator (env ping at session start)
→ problem-parser → problem-classifier → related-paper-analyzer
                                                ↓ Gate G1: PROBLEM_PARSED
→ symbol-table-builder + model-assumptions-builder + data-auditor-cleaner
→ method-selector  (each candidate: ≤30-line PoC + feasibility number; failures archived)
                                                ↓ Gate G2: METHOD_VALIDATED (load-bearing)
→ model-code-analyzer
   ├── python-model-code-generator
   └── matlab-model-code-generator
→ code-reviewer (router)
   ├── python-code-reviewer  (must write review file with ≥ 5 pass items)
   └── matlab-code-reviewer  (must write review file with ≥ 5 pass items)
                                                ↓ Gate G3: CODE_REVIEWED
→ result-report-generator (experiment reports + final analysis; [REJECTED] archived)
→ robustness-checker → final-method-explainer
→ figure-table-planner → math-figure-generator (render_check enforced)
→ solution-package-builder  (emits frozen_numbers.json — immutable snapshot)
                                                ↓ Gate G4: RESULTS_FROZEN (load-bearing)
→ paper-section-writer  (word-count floor + ≥ 3 discussion dimensions per number)
                                                ↓ Gate G5: PAPER_SECTION_READY
→ paper-polisher → reference-manager
→ [Independent audit layer] consistency-auditor + completeness-auditor + quality-assurance-auditor
                                                ↓ Gate G6: AUDIT_LAYER_PASSED (all three audits PASS)
→ final assembly
```

The order and gates matter. The orchestrator holds these lines:

- Do not select methods before the problem is parsed (G1).
- Do not generate full code before a candidate has a runnable PoC (G2 — load-bearing; prevents "beautiful method that breaks at code stage").
- Do not call code "reviewed" without a `code/Qx/reviews/qx_<lang>_review.md` file containing ≥ 5 explicit pass items (G3).
- Do not write numerical paper claims before they are frozen into `frozen_numbers.json` (G4 — load-bearing; prevents "paper says K=30 but code says K=12" drift).
- Do not call a section drafted if it's below word-count floor or has fewer than 3 discussion dimensions per number (G5).
- Do not allow final assembly unless all three orthogonal auditors PASS (G6).

Method selection should not collapse into a single route too early. For each subquestion, `method-selector` compares 2–4 candidates; each must include a runnable ≤30-line PoC or Gate G2 blocks code generation.

For contests requiring MATLAB / 北太天元, set `implementation.target` to `matlab` and use conservative runtime notes such as `beita-tianyuan-compatible` and `avoid-heavy-toolboxes`.

The language split affects only code generation and code review. Downstream skills consume artifacts from `results/Qx/`, regardless of whether the scripts came from Python or MATLAB.

## Documents

- [Modeling Workflow](docs/modeling-workflow.md): how the skills are connected, including the Python / MATLAB code branches.
- [Problem Taxonomy](docs/problem-taxonomy.md): task types used by `problem-classifier`.
- [Method Selection Tree](docs/method-selection-tree.md): baseline, main model, and improvement choices.
- [Design Principles](docs/design-principles.md): project-level modeling and workflow principles.
- [Figure and Table Guidelines](docs/figure-table-guidelines.md): how to plan visual evidence.
- [Paper Writing Rules](docs/paper-writing-rules.md): rules for drafting paper sections from artifacts.
- [QA Checklist](docs/qa-checklist.md): final audit checklist before submission.

## Usage

Before starting the first conversation, it is recommended to send the corresponding initial prompt:

- English conversation: [Initial Prompt.md](Initial Prompt.md)![Attachment.tiff](../../../../Attachment.tiff)
- Chinese conversation: [Initial Prompt-zh.md](Initial Prompt-zh.md)![Attachment.tiff](../../../../Attachment.tiff)

### Claude Code

Clone this repository and open it in Claude Code. The main files are:

- `CLAUDE.md`: project-level rules
- `.claude/skills/<skill-name>/SKILL.md`: stage-specific skill definitions
- `templates/`: output format references for parsed problems, method plans, and other artifacts
- `examples/`: example workflows, still being updated

A reasonable first message:

```text
Read CLAUDE.md, then run workflow-orchestrator. Our contest problem is in workspace/problem/. Follow the stage order and do not skip steps.
```

### Codex

The idea is the same, but the entry files are different:

- `AGENTS.md`: project-level rules
- `.codex/skills/<skill-name>/SKILL.md`: Codex skill definitions

A reasonable first message:

```text
Read AGENTS.md, then start with workflow-orchestrator. Our contest problem is in workspace/problem/. Follow the stage order and do not skip steps.
```

### Suggested Workspace Structure

```text
project/
├── planning/                   # Global planning (parse, classification, symbol table, assumptions, dashboard)
├── methods/Qx/                 # Candidate methods + PoCs + iteration log + final method explanation
│   └── poc/                    # ≤30-line PoC scripts per candidate (Gate G2)
├── code/
│   ├── model-code-analyzer.md
│   ├── Qx/                     # Python code
│   │   └── reviews/qx_python_review.md   # ≥5 explicit pass items (Gate G3)
│   └── matlab/Qx/              # MATLAB code (parallel structure)
├── results/Qx/
│   ├── experiments/roundN/     # figures / tables / metrics / logs / run_summary.json
│   └── reports/                # experiment reports + final analysis + solution package + frozen_numbers.json (Gate G4)
├── robustness/Qx/              # robustness reports + figures
├── paper/
│   ├── sections/               # section drafts (word floor + ≥3 discussion dims, Gate G5)
│   ├── figures/                # Type 3 + Type 4 figures (render_check passed)
│   │   └── render_check.log
│   ├── audits/                 # independent audit layer outputs (Gate G6)
│   │   ├── cross_media_consistency_audit.md
│   │   ├── completeness_audit.md
│   │   ├── reference_audit.md
│   │   └── polish_report.md
│   ├── refs.bib  main.tex  qa_report.md
├── workspace/
│   ├── data_raw/               # raw data — read-only (settings.json denies writes)
│   ├── data_clean/             # cleaned data
│   └── archived/<Qx>/<method>_REJECTED_roundN/   # archived failed methods
└── scratch/                    # temporary exploration (not reproducible)
```

Rules:

1. `workspace/data_raw/` is read-only. `.claude/settings.json` denies writes via permissions.
2. Every number in `paper/sections/` must appear in `results/Qx/reports/frozen_numbers.json` — `consistency-auditor` cross-checks them.
3. `[REJECTED]` methods are auto-archived to `workspace/archived/`. The main tree only ever contains `[CHOSEN]` and `[BACKUP]`.
4. `frozen_numbers.json` is **immutable by convention**. To change a frozen number, walk the `unfreeze → modify → re-freeze` three-step (log the reason in `freeze_change_log.md`, then re-invoke `solution-package-builder`).

### **Example Prompts**

Starting a new contest task:

```text
Use workflow-orchestrator. Our problem is in workspace/problem/problem.pdf, and the data is in workspace/data_raw/. Start from problem-parser.
```

Resuming from an existing method plan:

```text
The method plan is in workspace/problem/method_plan.json, and the cleaned data is in workspace/data_clean/. Continue from model-code-generator.
```

Running a robustness check on existing results:

```text
Results are in workspace/results/. Use robustness-checker to compare them with baseline_results.csv. Do not rerun the main model.
```

## Current Status

This project is still in an early first version.

- The skill prompts will keep being adjusted as more contest problems are tested.
- The templates and JSON schemas are intentionally simple for now.
- The `examples/` directory is still mostly a placeholder. Complete end-to-end examples are planned next.

If you run a full workflow and find something broken or awkward, feel free to open an issue or contact me at [zjzhang0424@gmail.com](mailto:zjzhang0424@gmail.com)![Attachment.tiff](../../../../Attachment.tiff). Thank you.

## Roadmap

- Add complete examples for evaluation, prediction, optimization, and hybrid problems.
- Tighten the JSON schemas for problem parsing, method planning, figure planning, and QA reports.
- Add method cards for common model families, including AHP/TOPSIS, regression, time series, MIP/heuristics, graph models, and simulation.
- Improve the handoff between `code-reviewer` and `robustness-checker` to reduce duplicated checks.
- Welcome notes in `docs/` about common failure cases, such as data leakage, baseline drift, and figure-result mismatch.
- Improve compatibility with other AI coding and writing tools.

## Acknowledgments & References

This project has drawn inspiration and design patterns from the following excellent work:

- **[nature-skills](https://github.com/Yuan1z0825/nature-skills)** — A collection of Claude Code skills for producing academic work at *Nature*-journal standard. The `math-figure-generator` skill in this project adapts the `nature-figure` skill's philosophy of figure contracts, semantic color palettes, multi-panel information architecture, and SVG-first export policy. nature-skills is maintained by [Yuan1z0825](https://github.com/Yuan1z0825) and community contributors, released under the MIT License.
- **[figures4papers](https://github.com/ChenLiu-1996/figures4papers)** — The production plotting scripts behind nature-skills' `nature-figure`, published in *Nature Machine Intelligence* and top ML/bioinformatics venues.

Our thanks to the authors and contributors of these projects.

## License

MIT.

<p align="center">
  <img src="docs/assets/logo.svg" alt="MathModeling-skills" width="640"/>
</p>

<p align="center">
  <a href="./README.md"><b>English</b></a> ·
  <a href="./README-zh.md">简体中文</a> ·
  <a href="./CLAUDE.md">Project Rules</a> ·
  <a href="./Initial%20Prompt.md">Initial Prompt</a> ·
  <a href="#standard-workflow-with-gates-g1g6">Workflow</a> ·
  <a href="#skills-by-pipeline-stage">Skill index</a>
</p>

<p align="center">
  <img alt="License" src="https://img.shields.io/badge/license-MIT-2E9E44">
  <img alt="Skills" src="https://img.shields.io/badge/skills-26-1A6FC4">
  <img alt="Gates" src="https://img.shields.io/badge/gates-G1--G6-1A6FC4">
  <img alt="Auditors" src="https://img.shields.io/badge/independent%20auditors-3-2E9E44">
  <img alt="Claude Code" src="https://img.shields.io/badge/Claude%20Code-supported-E28E2C">
  <img alt="Codex" src="https://img.shields.io/badge/Codex-supported-E28E2C">
  <img alt="Contest" src="https://img.shields.io/badge/contest-MCM%20%7C%20CUMCM-7B5FD6">
</p>

---

> **One skill stack for math-modeling contests.** `MathModeling-skills` converts a raw contest brief into a delivery-ready paper by orchestrating **26 single-purpose skills** behind **6 explicit gates** and a **3-auditor independent layer**. Every number in the paper is traceable to an immutable frozen snapshot, every reviewer / auditor must leave a substantive artifact on disk, and no skill can self-declare "done".

## Why this exists

The real problems in modeling contests rarely come from "not knowing enough models". They come from **workflow drift**:

- the problem gets misread and the work solves a different question
- a complex model is picked before any baseline is tried
- code runs, but does not match the method described in the paper
- the paper claims a number that cannot be found in any script output
- "our model is better" is written without a baseline or a sensitivity check
- a beautiful figure does not actually defend any single claim
- a bug fix silently moves the paper's numbers without anyone noticing

`MathModeling-skills` is built around these failure modes. Each skill has one narrow job. Each gate forces the workflow to converge before advancing. Three independent auditors check what the workflow has produced. The user keeps modeling judgment; the system keeps the audit trail.

## What's different

| | Stage-sequential workflow | `MathModeling-skills` |
|---|---|---|
| **Advancing rule** | "current stage done, go next" | each gate has explicit `enter_condition / pass_criteria / fail_fallback`; failure marks downstream artifacts DIRTY |
| **Method→code boundary** | accepted on math elegance | each candidate must ship a runnable **≤30-line PoC + feasibility number** (Gate G2) |
| **Code review** | verbal "passed" | reviewer file on disk with **≥ 5 explicit pass items** at `code/Qx/reviews/qx_<lang>_review.md` (Gate G3) |
| **Results→paper boundary** | numbers re-read from latest results | numbers locked into immutable **`frozen_numbers.json`** with provenance; `unfreeze → modify → re-freeze` three-step required to change (Gate G4) |
| **Section quality** | "looks fine" | section word-count floor + **≥ 3 discussion dimensions per numerical result** (Gate G5) |
| **Figure quality** | visual review by author | `render_check_and_log()` enforces bbox / out-of-canvas / font-size before promotion to Type 3 paper figure |
| **Final approval** | one QA pass | **3 orthogonal auditors** (consistency / completeness / QA) must all PASS — any one fails, assembly blocked (Gate G6) |
| **Failed methods** | linger in main tree | `[REJECTED]` auto-archived to `workspace/archived/<Qx>/<method>_REJECTED_roundN/` |
| **User hard rules** | repeated in every prompt | persistent guardrails in `.claude/settings.json` (frozen / raw data deny; git push ask) |

## Project boundaries

- It does **not** write a complete contest paper in one click.
- It does **not** invent data, results, references, or figures when evidence is missing. Missing evidence is reported as a blocker.
- It does **not** write numerical claims before results exist.
- It does **not** claim a model is better without a baseline and robustness or sensitivity checks.
- It does **not** modify your raw data — `data_raw/` is read-only.
- It does **not** replace modeling judgment. The final method choice still belongs to the user.

## Standard workflow (with Gates G1–G6)

```text
workflow-orchestrator (env ping at session start)
 │
 ▼  problem-parser → problem-classifier → related-paper-analyzer
                                          [ Gate G1: PROBLEM_PARSED ]
 ▼  symbol-table-builder + model-assumptions-builder + data-auditor-cleaner
 ▼  method-selector   ── each candidate: ≤30-line PoC + feasibility number
                                          [ Gate G2: METHOD_VALIDATED  ★ load-bearing ]
 ▼  model-code-analyzer
        ├── python-model-code-generator
        └── matlab-model-code-generator
 ▼  code-reviewer (router)
        ├── python-code-reviewer   ── writes qx_python_review.md (≥5 pass items)
        └── matlab-code-reviewer   ── writes qx_matlab_review.md (≥5 pass items)
                                          [ Gate G3: CODE_REVIEWED ]
 ▼  result-report-generator   ── [REJECTED] methods auto-archived
 ▼  robustness-checker → final-method-explainer
 ▼  figure-table-planner → math-figure-generator   ── render_check enforced
 ▼  solution-package-builder   ── emits frozen_numbers.json (immutable)
                                          [ Gate G4: RESULTS_FROZEN   ★ load-bearing ]
 ▼  paper-section-writer   ── word-count floor + ≥3 discussion dimensions per number
                                          [ Gate G5: PAPER_SECTION_READY ]
 ▼  paper-polisher → reference-manager
 ▼  Independent audit layer (all three must PASS):
        ├── consistency-auditor    (cross-media numbers / files / symbols / parameters)
        ├── completeness-auditor   (every required *_review.md / *_audit.md on disk, ≥5 pass items)
        └── quality-assurance-auditor  (workflow completeness + 3 critical rules + anti-fabrication)
                                          [ Gate G6: AUDIT_LAYER_PASSED ]
 ▼  final assembly
```

The orchestrator holds these lines:

- **G1** — Do not select methods before the problem is parsed.
- **G2** — Do not generate full code before a candidate ships a runnable PoC + feasibility number. *Load-bearing: prevents "beautiful method that breaks at code stage".*
- **G3** — Do not call code "reviewed" without a `qx_<lang>_review.md` file containing ≥ 5 explicit pass items.
- **G4** — Do not write numerical paper claims before they are frozen into `frozen_numbers.json`. *Load-bearing: prevents "paper says K=30 but code says K=12" drift after a bug fix.*
- **G5** — Do not call a section drafted if it is below the word-count floor or has fewer than 3 discussion dimensions per number.
- **G6** — Do not allow final assembly unless **all three** orthogonal auditors PASS. A single auditor's ✅ is never sufficient.

## Skills by pipeline stage

The 26 skills cluster into 5 pipeline stages plus the audit layer. Each skill lists **What it does**, the **Gate** it serves, **Outputs**, and **Key rule**.

---

### Stage 1 — Setup & global planning

Lay the foundation that the whole pipeline reads from: the parsed problem, the per-subquestion task type, the literature signal, the unified symbol table, the global model assumptions, and the cleaned data. After this stage, every later skill has a single canonical source for "what does Qx ask" and "what is the data".

#### `workflow-orchestrator`
- **What it does** — Tracks per-question state and decides which skill runs next.
- **Gate** — Evaluates G1–G6 against `enter_condition / pass_criteria / fail_fallback`; pings the environment (Python / MATLAB / git) at session start.
- **Outputs** — `planning/progress_dashboard.md`; recommended-next-skill report.
- **Key rule** — Never approves final assembly unless all three independent auditors PASS.

#### `problem-parser`
- **What it does** — Parses the contest problem into goals / objects / constraints / data / outputs / subquestions.
- **Outputs** — `planning/parse/`.
- **Key rule** — Does not classify or pick methods — those belong to later skills.

#### `problem-classifier`
- **What it does** — Tags each subquestion with a primary task type (evaluation, prediction, optimization, mechanism, classification, graph, simulation, data analysis, or hybrid).
- **Outputs** — `planning/classification/`.
- **Key rule** — Uses primary + secondary type when a subquestion legitimately spans two task families.

#### `related-paper-analyzer`
- **What it does** — Collects and analyzes relevant papers, reports, and transferable methods before final method selection.
- **Outputs** — `workspace/papers/related_paper_analysis.md`.
- **Key rule** — Never fabricates citations or methods that no source actually used.

#### `symbol-table-builder`
- **What it does** — Builds a unified global symbol table covering every subquestion (decision variables, parameters, sets, indices, outputs, units).
- **Outputs** — `planning/symbol_table.md`.
- **Key rule** — Same meaning gets the same symbol across subquestions; different meanings must use different symbols.

#### `model-assumptions-builder`
- **What it does** — Extracts and organizes global model assumptions, separating **necessary** assumptions (model breaks without them) from **simplifying** ones (model is approximate without them).
- **Outputs** — `planning/model_assumptions.md`.
- **Key rule** — Every assumption is linked to a modeling need with a stated impact if violated.

#### `data-auditor-cleaner`
- **What it does** — Audits raw data, lists issues, produces cleaned data + a data report. Copy-only — `workspace/data_raw/` is never touched.
- **Outputs** — `workspace/data_clean/`, `workspace/data_clean/data_report.md`.
- **Key rule** — Raw data is read-only (enforced by `.claude/settings.json` permissions).

---

### Stage 2 — Method validation (Gate G2)

This is the **load-bearing** boundary between the modeler's idea and the programmer's code. Method selection without PoC is where contests fail: a method looks elegant on paper, then collapses on real data three days before deadline. Gate G2 forces every candidate to ship a runnable feasibility check before code generation can start.

#### `method-selector`
- **What it does** — Proposes 2–4 candidates per subquestion; for each, requires a runnable **≤ 30-line PoC** + a concrete feasibility number on a small slice of the actual cleaned data. Asks one most-load-bearing question to the modeler up front (granularity alignment).
- **Gate** — G2 (METHOD_VALIDATED). A candidate without PoC is not a candidate; it's a hypothesis.
- **Outputs** — `methods/Qx/qx_method_candidates.md` + `methods/Qx/poc/<candidate>_poc.py|.m`.
- **Key rule** — PoC failure → mark candidate `[REJECTED]` and move the PoC script to `workspace/archived/`. Main `methods/Qx/poc/` keeps only `[CHOSEN]` and `[BACKUP]`.

---

### Stage 3 — Code execution & review (Gate G3)

Translate the validated method into runnable code, run experiments, and verify the implementation against the method plan. Reviewer skills must leave a substantive review file on disk — verbal "passed" is treated as MISSING by `completeness-auditor`.

#### `model-code-analyzer`
- **What it does** — Plans the experiment folder structure (`experiments/roundN/`), the `run_summary.json` schema, and the file layout for each method, before any code is written.
- **Outputs** — `code/model-code-analyzer.md`.
- **Key rule** — All experiment outputs land under `results/Qx/experiments/roundN/{figures,tables,metrics,logs}/`.

#### `python-model-code-generator`
- **What it does** — Generates Python modeling code when `implementation.target = python`.
- **Outputs** — `code/Qx/*.py`.
- **Key rule** — Saves outputs to files (CSV / JSON / PNG), never relies on console only; sets `SEED = 2026` for reproducibility.

#### `matlab-model-code-generator`
- **What it does** — Generates MATLAB / 北太天元 compatible code when `implementation.target = matlab`. Avoids Live Scripts, App Designer, heavy toolboxes by default.
- **Outputs** — `code/matlab/Qx/*.m`.
- **Key rule** — Conservative MATLAB syntax; flags Optimization / Deep Learning / Parallel toolbox calls as compatibility risks.

#### `code-reviewer`
- **What it does** — Router that detects script language and hands the review to `python-code-reviewer` or `matlab-code-reviewer`.
- **Key rule** — Never writes a review itself; if both languages are present, routes both branches.

#### `python-code-reviewer`
- **What it does** — Reviews Python modeling code against the method plan; fixes path / saving / shape / leakage issues with surgical edits; never changes the math.
- **Gate** — G3 (CODE_REVIEWED).
- **Outputs** — `code/Qx/reviews/qx_python_review.md` with **≥ 5 explicit pass items** (file:line cited).
- **Key rule** — A pass list with fewer than 5 concrete items returns `status: not_run` — no padding with generic statements.

#### `matlab-code-reviewer`
- **What it does** — Reviews MATLAB / 北太天元 code against the method plan; flags toolbox / Live-Script / App-Designer dependencies; surgical fixes only.
- **Gate** — G3 (CODE_REVIEWED).
- **Outputs** — `code/matlab/Qx/reviews/qx_matlab_review.md` with **≥ 5 explicit pass items**.
- **Key rule** — Same as Python reviewer — no padding, no verbal "passed".

---

### Stage 4 — Results, robustness, figures, freeze (Gate G4)

Turn raw experiment outputs into a paper-ready package. The final step of this stage — `solution-package-builder` — emits the **immutable** `frozen_numbers.json`. From this moment on, every number that lands in the paper must be sourced from this snapshot. Bug fixes in the code after freeze require an explicit `unfreeze → modify → re-freeze` walk.

#### `result-report-generator`
- **What it does** — Produces a per-round experiment report comparing all methods, then (when the modeler locks the final method) a final result analysis. Marks each method `[CHOSEN]`, `[BACKUP]`, or `[REJECTED]`.
- **Outputs** — `results/Qx/experiments/roundN/qx_experiment_report_roundN.md`; `results/Qx/reports/qx_final_result_analysis.md`.
- **Key rule** — `[REJECTED]` methods are moved to `workspace/archived/<Qx>/<method>_REJECTED_roundN/`. Main tree stays clean.

#### `robustness-checker`
- **What it does** — Designs and runs robustness, sensitivity, error, and baseline-comparison checks. Reports stable vs fragile conclusions.
- **Outputs** — `robustness/Qx/qx_robustness_report.md` with ≥ 5 explicit pass items.
- **Key rule** — Scan ranges (parameter perturbation %, scenario count) are pre-set by problem type — not chosen by agent intuition.

#### `final-method-explainer`
- **What it does** — Helps the modeler document the final selected method end-to-end: assumptions, symbols, objective, constraints, solution procedure, evaluation.
- **Outputs** — `methods/Qx/qx_final_method_explanation.md`.
- **Key rule** — The paper's model description must align with this file — not with an early candidate from `qx_method_candidates.md`.

#### `figure-table-planner`
- **What it does** — Plans figures and tables, classifying each into Type 1 (diagnostic, modeler-only) / Type 2 (comparison) / Type 3 (paper) / Type 4 (appendix).
- **Outputs** — `methods/Qx/qx_figure_table_plan.md`.
- **Key rule** — Type 1 figures never reach the paper.

#### `math-figure-generator`
- **What it does** — Publication-quality matplotlib figures with semantic palettes, panel logic, and SVG-first export. Inherited from the `nature-figure` design philosophy.
- **Outputs** — `paper/figures/qx_*.svg` + `paper/figures/render_check.log`.
- **Key rule** — Every Type 3 / Type 4 figure must pass `render_check_and_log()` (bbox overlap / out-of-canvas / font size < 6.5pt). Failure blocks Type 3 promotion.

#### `solution-package-builder`
- **What it does** — Integrates the final method explanation + final result analysis + figure/table plan into a single writer-facing package. Then emits the immutable numerical snapshot.
- **Gate** — G4 (RESULTS_FROZEN).
- **Outputs** — `results/Qx/reports/qx_solution_package_for_writer.md` + **`results/Qx/reports/frozen_numbers.json`**.
- **Key rule** — `frozen_numbers.json` is never edited by hand. To change a frozen number, walk `unfreeze → modify → re-freeze` (log the reason in `freeze_change_log.md`, then re-invoke this skill).

---

### Stage 5 — Paper drafting & independent audit (Gates G5 + G6)

The writer drafts paper sections only from the solution package and the frozen snapshot — never from scattered raw results. The independent audit layer then runs three orthogonal checks before final assembly. **Any one auditor failing blocks the paper.**

#### `paper-section-writer`
- **What it does** — Drafts paper sections from validated artifacts. Refuses to write Qx's main sections until the three critical rules (final method explanation + final result analysis + solution package) are satisfied.
- **Gate** — G5 (PAPER_SECTION_READY).
- **Outputs** — `paper/sections/qx.tex` (or `.md`).
- **Key rule** — Every section meets its word-count floor (per section type); every numerical result has **≥ 3 discussion dimensions** out of {sensitivity, physical meaning, baseline comparison, cross-subquestion consistency, uncertainty}.

#### `paper-polisher`
- **What it does** — Polishes syntax, tense, hedging calibration, overclaim detection, and within-document formula consistency.
- **Outputs** — `paper/audits/polish_report.md`.
- **Key rule** — Hedging matches evidence strength (`demonstrate` → `suggest` → `may reflect`); flags absolute / unwarranted-causation / first-claim patterns.

#### `reference-manager`
- **What it does** — Generates BibTeX entries; cross-checks every citation against retrievable sources.
- **Outputs** — `paper/refs.bib` + `paper/audits/reference_audit.md`.
- **Key rule** — Fabricated citations are blocking issues; every entry must have a verifiable DOI / URL / source.

#### `consistency-auditor` *(audit layer 1/3)*
- **What it does** — Cross-media check: every numerical claim / file reference / symbol / parameter in `paper/sections/*.tex` must match the source of truth (preferring `frozen_numbers.json`, then solution package, then raw metrics).
- **Outputs** — `paper/audits/cross_media_consistency_audit.md` with ≥ 5 explicit pass items.
- **Key rule** — Number tolerance equals paper precision; any divergence is BLOCKING, never WARNING.

#### `completeness-auditor` *(audit layer 2/3)*
- **What it does** — Scans the workspace for every audit / review file that should exist per the registry; verifies each has ≥ 5 explicit pass items and that none is stale (frozen snapshots must be newer than their source files).
- **Outputs** — `paper/audits/completeness_audit.md` with ≥ 5 explicit pass items.
- **Key rule** — Verbal "passed" never counts. Missing on disk = MISSING = blocking.

#### `quality-assurance-auditor` *(audit layer 3/3)*
- **What it does** — Audits problem coverage, the three critical rules, model-result lineage, figure-claim traceability, anti-fabrication, and conclusion-vs-evidence proportionality.
- **Gate** — G6 (AUDIT_LAYER_PASSED) — only when all three auditors PASS.
- **Outputs** — `paper/qa_report.md` with ≥ 5 explicit pass items.
- **Key rule** — Never approves assembly unless `cross_media_consistency_audit.md` and `completeness_audit.md` are also PASSED. The three are orthogonal; one ✅ is never sufficient.

---

## Suggested workspace structure

```text
project/
├── planning/                   # Global planning (parse, classification, symbol table, assumptions, dashboard)
├── methods/Qx/                 # Candidates + iteration log + final method explanation + figure plan
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

**Workspace rules**

1. `workspace/data_raw/` is read-only. `.claude/settings.json` denies writes via permissions.
2. Every number in `paper/sections/` must appear in `results/Qx/reports/frozen_numbers.json` — `consistency-auditor` cross-checks them.
3. `[REJECTED]` methods are auto-archived. Main tree only ever contains `[CHOSEN]` and `[BACKUP]`.
4. `frozen_numbers.json` is **immutable by convention**. Use `unfreeze → modify → re-freeze` (log in `freeze_change_log.md`, re-invoke `solution-package-builder`).

## Quickstart

Send the initial prompt at the start of the first conversation:

- English: [Initial Prompt.md](Initial%20Prompt.md)
- Chinese: [Initial Prompt-zh.md](Initial%20Prompt-zh.md)

### Claude Code

```text
Read CLAUDE.md, then run workflow-orchestrator. Our contest problem is in workspace/problem/. Follow the gates in order and do not skip.
```

Entry files:
- `CLAUDE.md` — project rules (gates, audit layer, frozen numbers convention).
- `.claude/skills/<skill-name>/SKILL.md` — per-skill instructions.
- `.claude/settings.json` — hard-rule guardrails (deny writes to `data_raw/` and `frozen_numbers.json`; ask on `git push`).

### Codex

```text
Read AGENTS.md, then start with workflow-orchestrator. Our contest problem is in workspace/problem/. Follow the gates in order and do not skip.
```

Entry files:
- `AGENTS.md` — project rules (mirror of CLAUDE.md for Codex).
- `.codex/skills/<skill-name>/SKILL.md` — per-skill instructions (mirror of `.claude/skills/`).

### Example follow-up prompts

- **Resume mid-pipeline**: `Q2 has experiment report round1 done. Run workflow-orchestrator to decide whether to iterate or lock the method.`
- **Robustness only**: `Use robustness-checker for Q1. Inputs in results/Q1/reports/, baseline in results/Q1/experiments/round2/. Do not rerun the main model.`
- **Trigger the audit layer**: `All Qx sections drafted. Run consistency-auditor, then completeness-auditor, then quality-assurance-auditor.`

## Documents

- [CLAUDE.md](CLAUDE.md) — project-level rules: gates, the independent audit layer, the frozen-numbers convention.
- [AGENTS.md](AGENTS.md) — Codex-side mirror of the project rules.
- [Implementation targets](docs/implementation-targets.md) — `python` vs `matlab` decision and constraints.
- [MATLAB / 北太天元 guidelines](docs/matlab-beita-tianyuan-guidelines.md) — toolbox-avoidance rules for contest-friendly code.
- Per-skill instructions: [.claude/skills/](.claude/skills/) (Claude Code) or [.codex/skills/](.codex/skills/) (Codex).

## Status

This project is still in an early version. Skill prompts will keep being adjusted as more contest problems are tested. Templates and JSON schemas are deliberately simple. End-to-end examples are the next milestone.

If you run a full workflow and find something broken or awkward, please open an issue or email [zjzhang0424@gmail.com](mailto:zjzhang0424@gmail.com).

## Roadmap

- End-to-end examples for evaluation, prediction, optimization, and hybrid problems.
- Tighter JSON schemas for problem parsing, method planning, figure planning, audit reports.
- Method cards for common model families (AHP/TOPSIS, regression, time series, MIP/heuristics, graph models, simulation).
- Reduce duplicated work between `code-reviewer` and `robustness-checker`.
- Failure-mode notes in `docs/` (data leakage, baseline drift, figure-result mismatch).
- Compatibility with other AI coding/writing agents.

## Acknowledgments

- **[nature-skills](https://github.com/Yuan1z0825/nature-skills)** — A collection of Claude Code skills for academic work at *Nature*-journal standard. `math-figure-generator` adapts its figure-contract philosophy, semantic color palette, multi-panel information architecture, and SVG-first export policy. Maintained by [Yuan1z0825](https://github.com/Yuan1z0825), MIT License.
- **[figures4papers](https://github.com/ChenLiu-1996/figures4papers)** — Production plotting scripts behind `nature-figure`, published in *Nature Machine Intelligence* and top ML/bioinformatics venues.

## License

MIT.

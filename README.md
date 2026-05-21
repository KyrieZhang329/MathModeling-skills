<p align="center">
  <img src="docs/assets/logo.svg" alt="MathModeling-skills" width="640"/>
</p>

<p align="center">
  <a href="./README.md"><b>English</b></a> ·
  <a href="./README-zh.md">简体中文</a> ·
  <a href="./CLAUDE.md">Project Rules</a> ·
  <a href="./Initial%20Prompt.md">Initial Prompt</a> ·
  <a href="mailto:zjzhang0424@gmail.com">📧 Contact</a>
</p>

<p align="center">
  <img alt="License" src="https://img.shields.io/badge/license-MIT-2E9E44">
  <img alt="Skills" src="https://img.shields.io/badge/skills-26-1A6FC4">
  <img alt="Claude Code" src="https://img.shields.io/badge/Claude%20Code-supported-E28E2C">
  <img alt="Codex" src="https://img.shields.io/badge/Codex-supported-E28E2C">
</p>

---

> **One skill stack for math-modeling contests.** Turns a contest brief into a delivery-ready paper through **26 single-purpose skills** behind **6 explicit gates** and a **3-auditor independent layer**. Every paper number traces back to an immutable frozen snapshot; every reviewer must leave a substantive artifact on disk; no skill can self-declare "done".
>
> 📧 Bug report / feedback / contributions welcome at **[zjzhang0424@gmail.com](mailto:zjzhang0424@gmail.com)** — or open an issue.

## Why this exists

Modeling contests rarely fail because of "not knowing enough models". They fail because of **workflow drift**: misreading the problem, jumping past baselines, paper claims that no script output supports, bug fixes that silently move the paper's numbers. `MathModeling-skills` is structured to make each of these drift modes structurally impossible to hide.

## What's different

| | Typical stage-sequential workflow | `MathModeling-skills` |
|---|---|---|
| **Advancing rule** | "current stage done, go next" | each gate has `enter_condition / pass_criteria / fail_fallback`; failure marks downstream artifacts DIRTY |
| **Method → code boundary** | accepted on math elegance | every candidate ships **≤30-line PoC + feasibility number** (Gate G2) |
| **Code review** | verbal "passed" | review file with **≥ 5 explicit pass items** at `code/Qx/reviews/qx_<lang>_review.md` (Gate G3) |
| **Results → paper** | numbers re-read from latest results | immutable **`frozen_numbers.json`**; changing a frozen number requires an explicit `unfreeze → modify → re-freeze` (Gate G4) |
| **Final approval** | one QA pass | **3 orthogonal auditors** (consistency / completeness / QA) — any one fails, assembly blocked (Gate G6) |
| **Failed methods** | linger in main tree | `[REJECTED]` auto-archived to `workspace/archived/` |

## Standard workflow (Gates G1 – G6)

```text
workflow-orchestrator (env ping at session start)
 ▼  problem-parser → problem-classifier → related-paper-analyzer       [ G1: PROBLEM_PARSED ]
 ▼  symbol-table-builder + model-assumptions-builder + data-auditor-cleaner
 ▼  method-selector   ── each candidate ships ≤30-line PoC + number    [ G2: METHOD_VALIDATED  ★ ]
 ▼  model-code-analyzer → {python,matlab}-model-code-generator
 ▼  code-reviewer (router) → {python,matlab}-code-reviewer             [ G3: CODE_REVIEWED ]
 ▼  result-report-generator ([REJECTED] archived)
 ▼  robustness-checker → final-method-explainer
 ▼  figure-table-planner → math-figure-generator (render_check)
 ▼  solution-package-builder ── emits frozen_numbers.json              [ G4: RESULTS_FROZEN   ★ ]
 ▼  paper-section-writer (word floor + ≥3 discussion dims per number)  [ G5: PAPER_SECTION_READY ]
 ▼  paper-polisher → reference-manager
 ▼  Independent audit layer (all three must PASS):
       consistency-auditor · completeness-auditor · quality-assurance-auditor
                                                                       [ G6: AUDIT_LAYER_PASSED ]
 ▼  final assembly
```

★ = load-bearing convergence point. Most contest failures happen at G2 (beautiful method that breaks at code stage) and G4 (paper number drifts after a bug fix).

## Skills by pipeline stage

### Stage 1 — Setup & global planning

Lay the foundation every downstream skill reads from: the parsed problem, per-subquestion task type, literature signal, unified symbol table, global assumptions, cleaned data.

- **`workflow-orchestrator`** — Tracks per-question state, evaluates Gates G1–G6, pings the environment at session start.
- **`problem-parser`** — Parses goals / objects / constraints / data / outputs / subquestions → `planning/parse/`.
- **`problem-classifier`** — Tags each subquestion with a primary task type → `planning/classification/`.
- **`related-paper-analyzer`** — Collects relevant papers; never fabricates citations.
- **`symbol-table-builder`** — Unified global symbol table → `planning/symbol_table.md`.
- **`model-assumptions-builder`** — Necessary vs simplifying assumptions → `planning/model_assumptions.md`.
- **`data-auditor-cleaner`** — Audits raw data, produces cleaned copy + report; `data_raw/` is read-only.

### Stage 2 — Method validation (Gate G2 ★)

The load-bearing boundary between modeler's idea and programmer's code. Without PoC, contests collapse three days before deadline.

- **`method-selector`** — 2–4 candidates per subquestion; **each must ship a runnable ≤30-line PoC + concrete feasibility number on real data**. PoC failure → candidate marked `[REJECTED]` and archived. *Outputs:* `methods/Qx/qx_method_candidates.md` + `methods/Qx/poc/*`.

### Stage 3 — Code execution & review (Gate G3)

Translate validated method into code; verify against the method plan. Reviewer skills must leave a substantive review file — verbal "passed" doesn't count.

- **`model-code-analyzer`** — Plans `experiments/roundN/` structure + `run_summary.json` schema before code generation.
- **`python-model-code-generator`** — Generates `.py` when target is `python`; fixed `SEED = 2026`.
- **`matlab-model-code-generator`** — Generates MATLAB / 北太天元-compatible `.m`; no Live Scripts, no App Designer.
- **`code-reviewer`** — Routes to language-specific reviewer based on script type.
- **`python-code-reviewer`** — Must write `code/Qx/reviews/qx_python_review.md` with **≥ 5 explicit pass items** (file:line cited).
- **`matlab-code-reviewer`** — Same rule for `code/matlab/Qx/reviews/qx_matlab_review.md`.

### Stage 4 — Results, robustness, figures, freeze (Gate G4 ★)

Turn raw experiment outputs into a writer-ready package and emit the **immutable** numerical snapshot. After freeze, bug fixes require an explicit `unfreeze → modify → re-freeze` walk.

- **`result-report-generator`** — Per-round comparison + final analysis. Methods labeled `[CHOSEN] / [BACKUP] / [REJECTED]`; rejected ones moved to `workspace/archived/`.
- **`robustness-checker`** — Sensitivity / error / baseline checks → `robustness/Qx/qx_robustness_report.md` (≥ 5 pass items).
- **`final-method-explainer`** — End-to-end documentation of the locked method → `methods/Qx/qx_final_method_explanation.md`.
- **`figure-table-planner`** — Type 1 (diagnostic) / 2 (comparison) / 3 (paper) / 4 (appendix). Type 1 never reaches the paper.
- **`math-figure-generator`** — Publication-quality matplotlib; **mandatory `render_check_and_log()`** (bbox overlap / out-of-canvas / font < 6.5pt) before Type 3 promotion.
- **`solution-package-builder`** — Writer-facing package + **emits `results/Qx/reports/frozen_numbers.json`**. Never edit by hand.

### Stage 5 — Paper drafting & independent audit (Gates G5 + G6)

Writer drafts from the package + frozen snapshot only. The independent audit layer then runs three orthogonal checks before final assembly. **Any one auditor failing blocks the paper.**

- **`paper-section-writer`** — Drafts sections; enforces word-count floors + **≥ 3 discussion dimensions per numerical result** (sensitivity / physical / baseline / cross-Qx / uncertainty).
- **`paper-polisher`** — Tense, hedging calibration, overclaim detection, within-document formula consistency.
- **`reference-manager`** — BibTeX + verifiable-source check; fabricated citations are blocking.
- **`consistency-auditor`** — Cross-media: every paper claim must match `frozen_numbers.json` / files on disk / symbol table.
- **`completeness-auditor`** — Every required `*_review.md` / `*_audit.md` must exist on disk with ≥ 5 pass items; not stale.
- **`quality-assurance-auditor`** — Workflow completeness + three critical rules + anti-fabrication. **Final gate**: only approves assembly when consistency-auditor and completeness-auditor have also PASSED.

## Installation

This repo is meant to live **at the root of your contest project**, so `CLAUDE.md` / `AGENTS.md` / `.claude/settings.json` apply automatically and downstream skills can write under `methods/`, `code/`, `results/`, `paper/`. You can also install the skills globally if you prefer.

### Option A — Clone into your contest project (recommended)

```bash
# inside the folder that will hold methods/ code/ results/ paper/ ...
git clone https://github.com/KyrieZhang329/MathModeling-skills.git .skills-tmp
# move skill files into place
mv .skills-tmp/.claude .claude
mv .skills-tmp/.codex .codex
mv .skills-tmp/CLAUDE.md .
mv .skills-tmp/AGENTS.md .
mv .skills-tmp/docs ./skills-docs
rm -rf .skills-tmp
```

After cloning, open the folder in **Claude Code** or **Codex** — the 26 skills are discovered automatically. Then say:

```text
Read CLAUDE.md, then run workflow-orchestrator. Our contest problem is in workspace/problem/. Follow the gates in order and do not skip.
```

### Option B — Install skills globally (Claude Code)

```bash
git clone https://github.com/KyrieZhang329/MathModeling-skills.git
cd MathModeling-skills

# install all 26 skills as user-level Claude Code skills
mkdir -p ~/.claude/skills
for d in .claude/skills/*/; do
  cp -R "$d" ~/.claude/skills/
done
```

Restart Claude Code. The skills are now available in any project. Copy `CLAUDE.md` and `.claude/settings.json` into each new contest project so the gate rules and guardrails apply there too.

### Option C — Install skills globally (Codex)

```bash
git clone https://github.com/KyrieZhang329/MathModeling-skills.git
cd MathModeling-skills

# install all 26 skills as user-level Codex skills
mkdir -p ~/.codex/skills
for d in .codex/skills/*/; do
  cp -R "$d" ~/.codex/skills/
done
```

Restart Codex. Drop `AGENTS.md` into each new contest project to apply the same gate rules.

### Update after pulling new changes

```bash
cd MathModeling-skills && git pull
# then re-run the cp loop from Option B or C above to refresh installed skills
```

### Initial prompt

Send the matching initial prompt as the first message of a new conversation:

- English: [Initial Prompt.md](Initial%20Prompt.md)
- 中文: [Initial Prompt-zh.md](Initial%20Prompt-zh.md)

### Example follow-up prompts

- **Resume mid-pipeline:** `Q2 has experiment report round1 done. Let workflow-orchestrator decide whether to iterate or lock the method.`
- **Robustness only:** `Use robustness-checker for Q1. Inputs in results/Q1/reports/, baseline in results/Q1/experiments/round2/. Do not rerun the main model.`
- **Trigger the audit layer:** `All Qx sections drafted. Run consistency-auditor, then completeness-auditor, then quality-assurance-auditor.`

## Workspace structure

<details>
<summary>Click to expand</summary>

```text
project/
├── planning/                   # Parse, classification, symbol table, assumptions, dashboard
├── methods/Qx/                 # Candidates + iteration log + final method explanation + figure plan
│   └── poc/                    # ≤30-line PoC per candidate (Gate G2)
├── code/
│   ├── Qx/                     # Python code; reviews/qx_python_review.md (Gate G3)
│   └── matlab/Qx/              # MATLAB code (parallel structure)
├── results/Qx/
│   ├── experiments/roundN/     # figures / tables / metrics / logs / run_summary.json
│   └── reports/                # experiment + final analysis + solution package + frozen_numbers.json (Gate G4)
├── robustness/Qx/
├── paper/
│   ├── sections/               # word floor + ≥3 discussion dims (Gate G5)
│   ├── figures/                # Type 3 + Type 4 (render_check passed)
│   ├── audits/                 # cross_media / completeness / reference / polish (Gate G6)
│   ├── refs.bib  main.tex  qa_report.md
├── workspace/
│   ├── data_raw/               # read-only (settings.json deny)
│   ├── data_clean/
│   └── archived/<Qx>/<method>_REJECTED_roundN/
└── scratch/                    # temporary, not reproducible
```

**Rules:** `data_raw/` is read-only · every paper number must appear in `frozen_numbers.json` · `[REJECTED]` methods auto-archived · `frozen_numbers.json` is never edited by hand.

</details>

## Boundaries

Does not write a full paper one-click · does not invent missing data / results / references · does not write numerical claims before results exist · does not claim "our model is better" without baseline + robustness · does not touch raw data · does not replace modeling judgment.

## Documents

- [CLAUDE.md](CLAUDE.md) — project rules (gates, audit layer, frozen-numbers convention).
- [AGENTS.md](AGENTS.md) — Codex-side mirror.
- [docs/implementation-targets.md](docs/implementation-targets.md) — `python` vs `matlab` decision.
- [docs/matlab-beita-tianyuan-guidelines.md](docs/matlab-beita-tianyuan-guidelines.md) — contest-friendly MATLAB rules.
- Per-skill: [.claude/skills/](.claude/skills/) · [.codex/skills/](.codex/skills/).

## Contact & contributions

📧 **[zjzhang0424@gmail.com](mailto:zjzhang0424@gmail.com)** — bugs, feedback, contributions, or just to tell me you ran it through a real contest. Issues and PRs also welcome.

## Acknowledgments

- **[nature-skills](https://github.com/Yuan1z0825/nature-skills)** — `math-figure-generator` adapts the figure-contract philosophy, semantic color palette, multi-panel architecture, and SVG-first export from `nature-figure`. Maintained by [Yuan1z0825](https://github.com/Yuan1z0825), MIT.
- **[figures4papers](https://github.com/ChenLiu-1996/figures4papers)** — Production plotting scripts behind `nature-figure`.

## License

MIT.

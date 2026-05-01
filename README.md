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

- Keeps a unified modeling workflow, adds a related-paper analysis stage before method selection, and branches only at code generation and code review, for 16 total skills.
- Encourages a fixed process: read the problem, classify the subquestions, select methods, write code, check results, then draft the paper.
- Keeps intermediate outputs such as parsed problems, method plans, data reports, results, figures, and paper drafts, so that every important number can be traced back to its source.
- Uses the same skill set for Claude Code (`.claude/skills/`) and Codex (`.codex/skills/`).

## Project Boundaries

- It does not write a complete contest paper in one click.
- It does not invent data, results, references, or figures when evidence is missing. Missing evidence is reported as a blocker.
- It does not write numerical conclusions before results exist.
- It does not claim a model is better without a baseline and robustness or sensitivity checks.
- It does not modify your raw data. Cleaning should happen on copied data.
- It does not replace modeling judgment. The final choice of method still belongs to the user.

## Skills

| Skill                       | Purpose                                                      |
| --------------------------- | ------------------------------------------------------------ |
| `workflow-orchestrator`     | Tracks the current stage and decides which skill should be used next. |
| `problem-parser`            | Reads the problem and extracts goals, objects, constraints, data, outputs, and subquestions. |
| `problem-classifier`        | Classifies each subquestion into a task type, such as evaluation, prediction, or optimization. |
| `related-paper-analyzer`    | Collects and analyzes relevant papers and reference methods before final method selection. |
| `method-selector`           | Proposes 2-4 candidate modeling schemes for each subquestion and recommends one execution route. |
| `data-auditor-cleaner`      | Audits the data, reports issues, and produces cleaned data with a data report. |
| `model-code-generator`      | Router that reads `implementation.target` and hands off to the correct language-specific code generator. |
| `python-model-code-generator` | Generates Python modeling code when the implementation target is `python`. |
| `matlab-model-code-generator` | Generates MATLAB / 北太天元 compatible modeling code when the implementation target is `matlab`. |
| `code-reviewer`             | Router that inspects script families and hands off to the correct language-specific reviewer. |
| `python-code-reviewer`      | Reviews Python modeling code and keeps fixes minimal and traceable. |
| `matlab-code-reviewer`      | Reviews MATLAB / 北太天元 compatible modeling code and keeps fixes minimal and traceable. |
| `robustness-checker`        | Runs sensitivity analysis, error checks, and baseline comparisons. |
| `figure-table-planner`      | Plans the figures and tables that are actually needed, and clarifies what each one should show. |
| `paper-section-writer`      | Drafts paper sections using only existing results and validated artifacts. |
| `quality-assurance-auditor` | Performs the final check before submission, looking for inconsistencies, missing evidence, and unsupported claims. |

## Standard Workflow

```text
workflow-orchestrator
→ problem-parser
→ problem-classifier
→ related-paper-analyzer
→ method-selector
→ data-auditor-cleaner
→ model-code-generator (router)
   ├── python-model-code-generator
   └── matlab-model-code-generator
→ code-reviewer (router)
   ├── python-code-reviewer
   └── matlab-code-reviewer
→ robustness-checker
→ figure-table-planner
→ paper-section-writer
→ quality-assurance-auditor
→ workflow-orchestrator
```

The order matters. The orchestrator is expected to keep several basic gates:

- Do not select methods before the problem is parsed.
- Do not generate code before the method plan is reviewed.
- Do not write numerical claims in the paper before result files exist.
- Do not claim “our model is better” without a baseline and sensitivity analysis.
- Do not assemble the final paper before QA passes.

Method selection should not collapse into a single route too early. For each subquestion, `method-selector` should usually compare 2-4 candidate schemes before recommending one execution plan.

For contests requiring MATLAB / 北太天元, set `implementation.target` to `matlab` and use conservative runtime notes such as `beita-tianyuan-compatible` and `avoid-heavy-toolboxes`.

The language split affects only code generation and code review. Downstream skills should consume artifacts from `workspace/results/` and `workspace/figures/`, regardless of whether the scripts came from Python or MATLAB.

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

When using this repository for a real contest, keep all generated artifacts in a workspace separate from the skills repository:

```text
workspace/
├── problem/          # original problem statement and parsed problem JSON
├── data_raw/         # original data, unchanged
├── data_clean/       # cleaned data and data audit report
├── scripts/
│   ├── python/       # generated Python scripts
│   └── matlab/       # generated MATLAB / 北太天元 compatible scripts
├── results/          # tables, metrics, model outputs
├── figures/          # final figures
├── paper_sections/   # drafted paper sections
└── final_paper/      # assembled paper
```

Two rules are worth keeping:

1. Treat `data_raw/` as read-only. If data needs to be modified, copy it to `data_clean/` first.
2. Every section under `paper_sections/` should be traceable to files under `results/`, `figures/`, or `data_clean/`.
3. Keep `results/` and `figures/` language-neutral so downstream skills consume artifacts rather than runtime-specific code.

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

## License

MIT.

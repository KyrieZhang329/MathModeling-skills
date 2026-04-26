# MathModeling-skills

A set of Claude Code / Codex skills for working through mathematical modeling contests step by step.

## Why this exists

Most problems in a modeling contest don't come from not knowing models. They come from:

- misreading the problem and solving the wrong thing
- jumping to a fancy model before checking if a baseline works
- code that runs but doesn't actually match the method described in the paper
- numbers in the paper that don't match any script output
- claims like "our model is better" without a baseline or sensitivity check
- figures that look nice but don't support any specific conclusion

This repo is an attempt to make those mistakes harder to make. It splits the contest workflow into small skills, each with a narrow job, and asks the agent to finish one stage before starting the next.

It is not a paper generator. It is closer to a checklist that an agent (and you) follow together.

## What it does

- Breaks the contest workflow into 11 skills, each focused on one stage.
- Forces a fixed order: read the problem first, then classify, then pick methods, then code, then check, then write.
- Keeps intermediate outputs (parsed problem, method plan, data report, results, figures, draft sections) so you can go back and check where any number came from.
- Works in both Claude Code (`.claude/skills/`) and Codex (`.codex/skills/`) with the same skill set.

## What it does not do

- It will not write a full paper for you in one shot.
- It will not invent data, results, citations, or figures when evidence is missing. Missing evidence is reported as a blocker.
- It will not write numerical conclusions before there are actual results to back them up.
- It will not claim a model is better without a baseline and a robustness check.
- It will not modify your raw data. Cleaning happens on copies.
- It does not replace your modeling judgment. You still decide which method makes sense for the problem.

## Skills

| Skill | Purpose |
| --- | --- |
| `workflow-orchestrator` | Tracks where you are in the workflow and decides which skill to run next. |
| `problem-parser` | Reads the problem statement and pulls out goals, objects, constraints, data, outputs, and subquestions. |
| `problem-classifier` | Labels each subquestion with a problem type (evaluation, prediction, optimization, etc.). |
| `method-selector` | For each subquestion, picks a baseline, a main model, and optional improved variants. |
| `data-auditor-cleaner` | Audits the dataset, flags issues, and produces a cleaned copy with a short data report. |
| `model-code-generator` | Generates Python or MATLAB code from the agreed method plan. |
| `code-reviewer` | Reviews and fixes the generated code: bugs, mismatches with the plan, unnecessary complexity. |
| `robustness-checker` | Runs sensitivity, error, and baseline comparison checks on the results. |
| `figure-table-planner` | Plans which figures and tables are actually needed and what each one is supposed to show. |
| `paper-section-writer` | Drafts paper sections using only results that already exist. |
| `quality-assurance-auditor` | Final pass before submission: looks for inconsistencies, missing evidence, broken references. |

## Workflow

```text
workflow-orchestrator
→ problem-parser
→ problem-classifier
→ method-selector
→ data-auditor-cleaner
→ model-code-generator
→ code-reviewer
→ robustness-checker
→ figure-table-planner
→ paper-section-writer
→ quality-assurance-auditor
→ workflow-orchestrator
```

The order matters. A few rules the orchestrator tries to enforce:

- No method selection until the problem has been parsed.
- No code generation until the method plan is reviewed.
- No numerical claims in the paper until results exist on disk.
- No "our model is better" without a baseline result and a sensitivity check.
- No final assembly until QA has passed.

If you want to skip a step on purpose, that's fine, but it should be a deliberate choice rather than an accident.

## Usage

Before starting your first conversation, it is recommended to send the corresponding initial prompt:

- English conversation: [Initial Prompt.md](Initial%20Prompt.md)
- Chinese conversation: [Initial Prompt-zh.md](Initial%20Prompt-zh.md)

### Claude Code

Clone the repo and open it in Claude Code. The relevant files:

- `CLAUDE.md` — project-level rules.
- `.claude/skills/<skill-name>/SKILL.md` — the skill definitions.
- `templates/` — output shapes for parsed problems, method plans, etc.
- `examples/` — example runs (work in progress).

A reasonable first message:

```text
Read CLAUDE.md, then run workflow-orchestrator. The contest problem is in workspace/problem/. Don't skip stages.
```

### Codex

Same idea, just different entry points:

- `AGENTS.md` for project-level rules.
- `.codex/skills/<skill-name>/SKILL.md` for the skill set.

First message:

```text
Read AGENTS.md, then start with workflow-orchestrator. The contest problem is in workspace/problem/. Follow the stage order and do not skip stages.
```

### Suggested workspace

When you actually use this on a contest, keep generated files in a workspace separate from the skills repo:

```text
workspace/
├── problem/          # original problem statement, parsed problem JSON
├── data_raw/         # original data, never modified
├── data_clean/       # cleaned data + data audit report
├── scripts/          # generated Python / MATLAB code
├── results/          # tables, metrics, model outputs
├── figures/          # final figures
├── paper_sections/   # drafted sections
└── final_paper/      # assembled paper
```

Two things worth taking seriously:

1. `data_raw/` is read-only. If you need to modify data, copy it to `data_clean/` first.
2. Anything in `paper_sections/` should be traceable back to a file under `results/`, `figures/`, or `data_clean/`. If it isn't, something is wrong.

### Example prompts

Starting a new contest:

```text
Use workflow-orchestrator. Problem is in workspace/problem/problem.pdf, data is in workspace/data_raw/. Begin with problem-parser.
```

Resuming partway through:

```text
The method plan is in workspace/problem/method_plan.json and cleaned data is in workspace/data_clean/. Continue from model-code-generator.
```

Asking for a robustness pass on existing results:

```text
Results are in workspace/results/. Run robustness-checker against the baseline in baseline_results.csv. Don't rerun the main model.
```

## Current status

Still in an early first version, so some parts are intentionally rough:

- The skill prompts will keep evolving as more contest problems are run through them.
- Templates and JSON schemas are deliberately minimal right now.
- The `examples/` directory is mostly a placeholder. Real worked examples are next.

If you run a full workflow and spot problems, feel free to open an issue, or send details directly to [zjzhang0424@gmail.com](mailto:zjzhang0424@gmail.com). Thank you.

## Roadmap

- Add complete worked examples for evaluation, prediction, optimization, and hybrid problem types.
- Tighten JSON schemas for problem parsing, method planning, figure planning, and QA reports.
- Add method cards for common model families (AHP/TOPSIS, regression families, time series, MIP/heuristics, graph models, simulation).
- Improve the handoff between `code-reviewer` and `robustness-checker` to avoid duplicate checks.
- Contributions to `docs/` are welcome for common failure notes (data leakage, baseline drift, figure-result mismatch, and similar cases).

## License

MIT.

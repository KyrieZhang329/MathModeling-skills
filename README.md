# MathModeling-skills

Reusable Claude Code and Codex skills for mathematical modeling contests.

## What this repository is

- A lightweight open-source scaffold for contest-oriented agent skills.
- A place for reusable workflows, templates, and documentation placeholders.
- A foundation for future skills covering problem analysis, modeling, coding, writing, and QA.

## What this repository is not

- Not a collection of finished modeling algorithms.
- Not a contest solution generator with heavy tooling.
- Not a full tutorial or a benchmark dataset repository.

## Skills

- `workflow-orchestrator`: Manage the overall contest workflow and route the next step.
- `problem-parser`: Extract goals, objects, constraints, data, outputs, and subquestions.
- `problem-classifier`: Map subquestions to common modeling task types.
- `method-selector`: Choose baseline, main, and improved methods for each subproblem.
- `data-auditor-cleaner`: Prepare raw contest data for modeling use.
- `model-code-generator`: Turn a validated method plan into executable code.
- `code-reviewer`: Review and simplify modeling code before deeper checks.
- `robustness-checker`: Plan and run sensitivity, error, and baseline checks.
- `figure-table-planner`: Design figures and tables that support the argument.
- `paper-section-writer`: Draft paper sections from validated artifacts only.
- `quality-assurance-auditor`: Perform the final solution audit before submission.

Detailed skill instructions will be added later.

## Repository structure

```text
MathModeling-skills/
├── docs/
├── templates/
├── examples/
├── .claude/skills/
└── .codex/skills/
```

## Using with Claude Code

- Place repository-level guidance in `CLAUDE.md`.
- Load a matching skill when a contest task reaches that workflow stage.
- Keep changes minimal and move step by step through the workflow.

## Using with Codex

- Use `AGENTS.md` as the project rule file for agentic coding tools.
- Start from the workflow skill, then hand off to specialized skills as needed.
- Use templates and docs as placeholders until detailed instructions are added.

## Roadmap

- Expand placeholder skills into practical contest workflows.
- Refine shared templates and schemas for reusable outputs.
- Add sharper QA and writing guidance after the core workflow stabilizes.

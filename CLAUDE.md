# Core Philosophy

- Start from goals, objects, constraints, data, outputs, variables, relationships, and checkable conclusions.
- Do not start from model names.
- Separate assumptions from validated conclusions at every stage.

# Workflow Discipline

- Follow the full workflow in order instead of skipping ahead.
- Keep file edits minimal and traceable.
- Do not overwrite validated artifacts casually.
- Only generate code from a validated method plan.

# Workflow Gates

- Do not select methods before the problem is parsed.
- Do not generate code before the method plan is validated.
- Do not write numerical paper claims before results exist.
- Do not assemble the final paper before QA passes.

# Workspace Convention

- `workspace/problem/`: problem statement, parsed notes, and task breakdowns.
- `workspace/data_raw/`: untouched source data from the contest.
- `workspace/data_clean/`: cleaned data prepared for modeling.
- `workspace/scripts/`: minimal modeling scripts derived from validated plans.
- `workspace/results/`: intermediate and final numeric outputs.
- `workspace/figures/`: planned and generated figures for the paper.
- `workspace/paper_sections/`: section drafts built from validated artifacts.
- `workspace/final_paper/`: assembled final submission materials.

# Workspace Artifact Contract

- Problem parses go under `workspace/problem/`.
- Raw data must remain untouched under `workspace/data_raw/`.
- Cleaned data goes under `workspace/data_clean/`.
- Generated scripts go under `workspace/scripts/`.
- Numeric outputs and reports go under `workspace/results/`.
- Figures go under `workspace/figures/`.
- Paper section drafts go under `workspace/paper_sections/`.
- Final assembled materials go under `workspace/final_paper/`.

# Modeling Rules

- Parse first, classify second, select methods third.
- Match method choice to problem type, data, and contest limits.
- Every main model must have a baseline.
- Do not choose complex models only because they look advanced.
- Do not invent missing data, results, or references.

# Coding Rules

- Make surgical edits.
- Preserve existing style.
- Make the smallest necessary change when reading or editing files.
- Keep code readable and easy to audit.
- Do not overwrite validated artifacts unless explicitly repairing them.
- When modifying generated code, explain what changed and how to verify it.

# Paper Writing Rules

- Write only from available problem analysis, model plans, results, figures, and robustness checks.
- Keep claims proportional to evidence.
- Do not write numerical claims before results exist.
- Avoid filler prose and unsupported conclusions.

# Verification and QA

- Re-check assumptions, outputs, and narrative consistency before handoff.
- Robustness or sensitivity checks are required before final delivery.
- Final delivery requires QA approval.
- Mark blocking issues clearly.
- Return control to the workflow instead of improvising around unresolved problems.

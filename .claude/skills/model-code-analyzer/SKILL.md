---
name: model-code-analyzer
description: Translate a validated method plan into language-neutral code logic, folder layout, and handoff notes before Python or MATLAB code generation.
license: MIT
---

# Purpose

Translate validated mathematical modeling plans into executable code thinking.

This skill does not write runnable code itself. It turns the selected modeling route into a language-neutral coding plan: what each subquestion needs, which data enters each stage, which intermediate artifacts should be saved, how code should be split by subquestion, and which language-specific generator should implement the plan next.

# When to use

Use this skill:

- After `data-auditor-cleaner` has confirmed that required data is ready or ready with documented warnings.
- After `method-selector` has produced a validated method plan.
- Before `python-model-code-generator` or `matlab-model-code-generator`.
- When the team needs a code-thinking document under `workspace/code/` before writing executable scripts.

# Preconditions

The following should already exist or be provided:

- A validated method plan.
- An `implementation.target` field with value `python` or `matlab`.
- Cleaned data under `workspace/data/data_clean/`, if data is required.
- A data report at `workspace/data/data_report.md`, if data is used.

# Inputs

Use or request:

- `workspace/problem/method-selector/method_plan.json`
- `workspace/data/data_report.md`, if available
- cleaned data under `workspace/data/data_clean/`
- implementation target and runtime notes
- expected result outputs and figure outputs

# Workflow

1. Read the validated method plan.
   - Work subquestion by subquestion.
   - Preserve baseline, main model, and optional improved model roles.
   - Do not change the selected method plan.

2. Translate the mathematical plan into code logic.
   - Identify required inputs for each subquestion.
   - Identify calculation order, intermediate values, and saved artifacts.
   - Identify which outputs belong in `workspace/results/` and which figures belong in `workspace/figures/`.

3. Define the code layout.
   - Recommend one code folder per subquestion.
   - For Python, plan under `workspace/code/python/Q1/`, `workspace/code/python/Q2/`, and so on.
   - For MATLAB, plan under `workspace/code/matlab/Q1/`, `workspace/code/matlab/Q2/`, and so on.

4. Write the code-thinking document.
   - Save the analysis under `workspace/code/model-code-analyzer.md`.
   - Keep the document language-neutral.
   - Make the handoff usable by the language-specific code generator.

5. Hand off to the language-specific generator.
   - Route to `python-model-code-generator` when `implementation.target = python`.
   - Route to `matlab-model-code-generator` when `implementation.target = matlab`.

# Outputs

Produce one Markdown planning artifact:

- `workspace/code/model-code-analyzer.md`

The document should include:

- implementation target
- runtime notes
- per-subquestion code logic
- expected input data
- expected code folders
- expected result artifacts
- expected figure artifacts
- recommended next skill

# Output format

Write a concise Markdown document under `workspace/code/model-code-analyzer.md`.

The document should explain, for each subquestion:

- what the code must read
- what the code must compute
- what baseline and main outputs must be saved
- which files should be created under `workspace/code/`
- which outputs should be saved under `workspace/results/` and `workspace/figures/`

# Rules

- Do not write executable Python or MATLAB code in this skill.
- Do not change the validated method plan.
- Do not fabricate data, scripts, results, or figures.
- Do not bypass language-specific code generators.
- Keep the code plan minimal, traceable, and easy to review.

# Handoff

After writing `workspace/code/model-code-analyzer.md`, hand off to:

- `python-model-code-generator` if `implementation.target = python`
- `matlab-model-code-generator` if `implementation.target = matlab`

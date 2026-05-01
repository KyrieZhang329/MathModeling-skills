---
name: code-reviewer
description: Detect the language of generated modeling code and route review work to the Python or MATLAB reviewer without producing a separate saved artifact.
license: MIT
---

# Purpose

Route existing modeling code to the correct language-specific reviewer.

This skill does not do deep Python or MATLAB review itself. It only checks `implementation.target`, inspects script extensions and code folders, and decides whether review should be handled by `python-code-reviewer` or `matlab-code-reviewer`.

# When to use

Use this skill:

- After `python-model-code-generator` or `matlab-model-code-generator` has produced code.
- When the team needs to confirm whether the current code should be reviewed as Python or MATLAB.
- Before `robustness-checker`.

# Preconditions

The following should already exist or be provided:

- A validated method plan with `implementation.target`.
- Generated code under `workspace/code/python/` or `workspace/code/matlab/`.
- Expected result and figure output paths.

# Inputs

Use or request:

- `workspace/problem/method-selector/method_plan.json`
- generated code under `workspace/code/python/`
- generated code under `workspace/code/matlab/`
- runtime notes
- expected result and figure paths

# Workflow

1. Read `implementation.target`.
   - Confirm that the target is `python` or `matlab`.

2. Inspect the available code files.
   - Route `.py` files to `python-code-reviewer`.
   - Route `.m` files to `matlab-code-reviewer`.
   - If the project is explicitly mixed-language, route each family separately.

3. Check for target conflicts.
   - Stop if the detected language conflicts with `implementation.target`.
   - Stop if the code layout is too ambiguous to review safely.

4. Hand off to the correct reviewer.
   - Do not create a saved router report under `workspace/`.
   - Do not perform deep language-specific review inside this skill.

# Outputs

This router does not create a persisted output artifact under `workspace/`.

Its only job is to choose the next reviewer in the current handoff.

# Rules

- Do not perform deep code review here.
- Do not modify code here.
- Do not create a separate report file here.
- Do not ignore conflicts between script language and `implementation.target`.

# Handoff

Route to:

- `python-code-reviewer` for Python code
- `matlab-code-reviewer` for MATLAB / 北太天元 compatible code

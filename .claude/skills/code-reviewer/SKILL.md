---
name: code-reviewer
description: Detect the language of generated modeling code and route review work to the Python or MATLAB reviewer without producing a separate saved artifact.
license: MIT
---

# Purpose

Route existing modeling code to the correct language-specific reviewer.

This skill does not do deep Python or MATLAB review itself. It only checks `implementation.target`, inspects script extensions and code folders (`code/Qx/` for Python, `code/matlab/Qx/` for MATLAB), and decides whether review should be handled by `python-code-reviewer` or `matlab-code-reviewer`.

It also verifies that generated code follows the expected `experiments/roundN/` output conventions before routing.

# When to use

Use this skill:

- After `python-model-code-generator` or `matlab-model-code-generator` has produced code.
- When the team needs to confirm whether the current code should be reviewed as Python or MATLAB.
- Before `result-report-generator` and `robustness-checker`.

# Preconditions

The following should already exist or be provided:

- A candidate method pool with `implementation.target` specified.
- Generated code under `code/Qx/` (Python) or `code/matlab/Qx/` (MATLAB).
- The code thinking document `code/model-code-analyzer.md`.
- Expected result and figure output paths under `results/Qx/experiments/roundN/`.

# Inputs

Use or request:

- `methods/Qx/qx_method_candidates.md` — candidate method pool.
- `code/model-code-analyzer.md` — code thinking document.
- Generated code under `code/Qx/` (Python `.py` files).
- Generated code under `code/matlab/Qx/` (MATLAB `.m` files).
- The `README.md` in each code folder.
- Runtime notes from the code generator.
- Expected result and figure paths.

# Workflow

1. Read `implementation.target` from the code thinking document or candidate pool.
   - Confirm that the target is `python` or `matlab`.

2. Inspect the available code files.
   - Check under `code/Qx/` for `.py` files → route to `python-code-reviewer`.
   - Check under `code/matlab/Qx/` for `.m` files → route to `matlab-code-reviewer`.
   - If the project is explicitly mixed-language (rare), route each family separately.

3. Verify output structure conventions.
   - Confirm scripts write to `results/Qx/experiments/roundN/tables/`, `figures/`, `metrics/`, `logs/`.
   - Confirm `run_summary.json` generation is implemented.
   - If output paths do not follow the expected structure, flag this to the language-specific reviewer.

4. Check for target conflicts.
   - Stop if the detected language conflicts with `implementation.target`.
   - Stop if the code layout is too ambiguous to review safely.

5. Hand off to the correct reviewer.
   - Do not create a saved router report under `workspace/`.
   - Do not perform deep language-specific review inside this skill.

# Outputs

This router does not create a persisted output artifact under `workspace/`.

Its only job is to choose the next reviewer and pass relevant context.

# Code structure verification (light checks before routing)

Before routing, verify at least:

- Code files exist in the expected language-specific folder (`code/Qx/` for Python, `code/matlab/Qx/` for MATLAB).
- Each code folder has a `README.md` with run instructions.
- Script file names suggest mapping to candidate methods (e.g., `q1_m1_baseline.py` maps to M1).
- Output paths in scripts point to `results/Qx/experiments/roundN/` (not old `workspace/results/` paths).

If any of these checks fail, pass the findings to the language-specific reviewer as pre-identified issues.

# Rules

- Do not perform deep code review here.
- Do not modify code here.
- Do not create a separate report file here.
- Do not ignore conflicts between script language and `implementation.target`.
- Verify output conventions match `experiments/roundN/` structure before routing.
- Pass pre-identified issues to the language-specific reviewer.

# Handoff

Route to:

- `python-code-reviewer` for Python code under `code/Qx/`
- `matlab-code-reviewer` for MATLAB / 北太天元 compatible code under `code/matlab/Qx/`

The handoff should include:
- Paths to all scripts to review.
- Code folder `README.md` paths.
- Candidate method pool path.
- Expected `experiments/roundN/` output structure.
- Any pre-identified issues (output path mismatches, missing README, etc.).
- Runtime notes and compatibility constraints.

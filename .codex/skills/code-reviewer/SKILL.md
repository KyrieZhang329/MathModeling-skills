---
name: code-reviewer
description: Route generated modeling scripts to Python or MATLAB code review skills based on implementation target and script language.
license: MIT
---

# Purpose

Route generated modeling scripts to the correct language-specific code reviewer.

This skill reads the validated method plan, checks `implementation.target`, inspects available script files, and hands off review work to either `python-code-reviewer` or `matlab-code-reviewer`.

This skill does not perform deep language-specific code review itself. It only decides which reviewer should inspect which scripts.

# When to use

Use this skill:

- After `python-model-code-generator` or `matlab-model-code-generator` has generated scripts.
- When generated scripts need to be reviewed before results are trusted.
- When the workflow reaches the code review stage.
- When script language may be Python, MATLAB, or a clearly specified mixed-language workflow.
- When contest requirements specify MATLAB / 北太天元 compatibility.

# Preconditions

The following should already exist or be provided:

- A validated method plan.
- An `implementation` object in the method plan.
- Generated scripts under a language-specific scripts directory.
- Cleaned data or approved non-tabular inputs.
- Expected result and figure output paths.
- Run instructions or intended script order.

The method plan should contain:

```json
{
  "implementation": {
    "target": "matlab",
    "runtime_notes": [
      "beita-tianyuan-compatible",
      "avoid-heavy-toolboxes"
    ]
  }
}
```

Expected script locations:

```text
workspace/scripts/python/
workspace/scripts/matlab/
```

Expected shared output locations:

```text
workspace/results/
workspace/figures/
```

# Inputs

Use or request:

- `workspace/problem/method_plan.json`, if available.
- generated Python scripts under `workspace/scripts/python/`
- generated MATLAB scripts under `workspace/scripts/matlab/`
- data audit report, if available
- cleaned data paths
- expected result paths
- expected figure paths
- run instructions
- runtime notes
- user-provided contest language requirements if relevant

# Workflow

1. Read the method plan.
   - Confirm that it is validated.
   - Confirm that it includes `implementation.target`.
   - Confirm that the target is either `python` or `matlab`.
2. Inspect generated scripts.
   - Identify `.py` scripts.
   - Identify `.m` scripts.
   - Check whether scripts are stored under the expected language-specific directory.
   - Check whether script types match the implementation target.
3. Resolve the review route.
   - If `implementation.target = python`, route Python scripts to `python-code-reviewer`.
   - If `implementation.target = matlab`, route MATLAB scripts to `matlab-code-reviewer`.
   - If script files conflict with the implementation target, stop and report a blocker.
   - If a mixed-language workflow is explicitly specified, route each script family separately.
4. Prepare handoff context.
   - Include script paths.
   - Include method plan path.
   - Include cleaned data paths.
   - Include expected output paths.
   - Include runtime notes.
   - Include known implementation risks.
5. Hand off to the language-specific reviewer.
   - Do not perform language-specific review inside this router.
   - Do not modify scripts inside this router.
   - Do not change the method plan.
   - Do not invent outputs or assume scripts are correct.

# Outputs

Produce a routing decision containing:

- `implementation_target`
- `detected_script_languages`
- `script_groups`
- `next_skill`
- `handoff_context`
- `blocked_items`, if routing cannot proceed
- `recommended_next_action`, if blocked

# Output format

Prefer this JSON-compatible structure:

```json
{
  "code_review_route": {
    "implementation_target": "matlab",
    "detected_script_languages": [
      "matlab"
    ],
    "script_groups": {
      "matlab": [
        "workspace/scripts/matlab/q1_main.m",
        "workspace/scripts/matlab/q1_baseline.m"
      ]
    },
    "next_skill": "matlab-code-reviewer",
    "handoff_context": {
      "method_plan": "workspace/problem/method_plan.json",
      "cleaned_data": [
        "workspace/data_clean/clean_data.csv"
      ],
      "expected_results_dir": "workspace/results/",
      "expected_figures_dir": "workspace/figures/",
      "runtime_notes": [
        "beita-tianyuan-compatible",
        "avoid-heavy-toolboxes"
      ]
    }
  }
}
```

If blocked, use:

```json
{
  "code_review_route": {
    "status": "blocked",
    "blocked_items": [
      "implementation.target is matlab, but only Python scripts were found."
    ],
    "recommended_next_skill": "model-code-generator",
    "recommended_next_action": "Regenerate scripts for the MATLAB target or update the method plan if Python is intended."
  }
}
```

# Routing rules

- `.py` scripts route to `python-code-reviewer`.
- `.m` scripts route to `matlab-code-reviewer`.
- `implementation.target = python` should normally correspond to `.py` scripts under `workspace/scripts/python/`.
- `implementation.target = matlab` should normally correspond to `.m` scripts under `workspace/scripts/matlab/`.
- MATLAB / 北太天元 compatibility is handled by `matlab-code-reviewer`.
- Do not treat 北太天元 as a separate third implementation target.
- Do not route `.m` scripts to the Python reviewer.
- Do not route `.py` scripts to the MATLAB reviewer.
- Do not perform mixed-language routing unless the method plan explicitly describes a mixed-language workflow.
- If the method plan and script files conflict, stop and report a blocker.

# Target conventions

## Python review route

Route to `python-code-reviewer` when:

- `implementation.target = python`
- generated scripts are `.py`
- scripts are under `workspace/scripts/python/`
- Python was explicitly requested and allowed by contest rules

Expected review scope:

- imports
- relative paths
- pandas / numpy shape consistency
- column names and data mapping
- random seeds
- leakage risks
- result saving
- figure saving
- method-plan consistency

## MATLAB / 北太天元 review route

Route to `matlab-code-reviewer` when:

- `implementation.target = matlab`
- generated scripts are `.m`
- scripts are under `workspace/scripts/matlab/`
- contest requires MATLAB / 北太天元
- runtime notes include compatibility requirements

Expected review scope:

- `.m` script runnability
- MATLAB / 北太天元 compatibility
- 1-based indexing
- row and column vector consistency
- matrix dimensions
- `readtable` / `writetable` usage
- `rng` usage
- toolbox dependency risk
- result saving
- figure saving
- method-plan consistency

# Rules

- Do not perform deep language-specific code review.
- Do not modify code inside this router.
- Do not change the method plan.
- Do not change the implementation target.
- Do not fabricate script paths, results, figures, metrics, or logs.
- Do not approve code as correct.
- Do not route scripts to a reviewer if the target is ambiguous.
- Do not ignore conflicts between target and script language.
- Do not bypass language-specific reviewers.
- Preserve workflow gates.

# Verification

Before handoff, verify:

- The method plan exists or is provided.
- `implementation.target` is present.
- The target is either `python` or `matlab`.
- Generated scripts exist or are explicitly listed.
- Script extensions match the intended target.
- Script directories are language-specific where practical:
  - `workspace/scripts/python/`
  - `workspace/scripts/matlab/`
- Expected result and figure directories remain language-neutral:
  - `workspace/results/`
  - `workspace/figures/`
- Runtime notes are preserved.
- The next skill is correct.

# Failure modes

Stop and report a blocker if:

- The method plan is missing.
- `implementation.target` is missing.
- `implementation.target` is not `python` or `matlab`.
- No scripts are available to review.
- Script language conflicts with the implementation target.
- Scripts are mixed-language but the method plan does not allow mixed-language workflow.
- Generated scripts cannot be linked to any subquestion.
- Review routing would require changing the modeling method.
- Required cleaned data is missing.

# Stop conditions

This skill must stop instead of guessing when:

- The implementation target is ambiguous.
- Script extensions are inconsistent with the target.
- Contest rules require MATLAB / 北太天元 but only Python scripts exist.
- The method plan says Python but only MATLAB scripts exist.
- A mixed-language workflow is present but undocumented.
- Continuing would hide a code-generation or method-plan error.

When stopping, output:

- blocker
- why it matters
- affected scripts
- affected method plan item
- recommended repair skill
- next action

# Handoff

If routing to Python review, hand off to:

```
python-code-reviewer
```

Include:

- Python script paths
- method plan
- cleaned data paths
- expected result paths
- expected figure paths
- runtime notes
- run instructions
- known implementation risks

If routing to MATLAB review, hand off to:

```
matlab-code-reviewer
```

Include:

- MATLAB script paths
- method plan
- cleaned data paths
- expected result paths
- expected figure paths
- runtime notes
- MATLAB / 北太天元 compatibility requirements
- run instructions
- known implementation risks

If blocked, hand back to:

- `model-code-generator` for missing or wrong script generation
- `method-selector` for target conflict
- `data-auditor-cleaner` for missing cleaned data
- `workflow-orchestrator` for unresolved mixed-language routing

# Examples

## Example 1: Route MATLAB scripts to MATLAB reviewer

Input state:

- `implementation.target = matlab`
- scripts exist under `workspace/scripts/matlab/`

Output:

```json
{
  "code_review_route": {
    "implementation_target": "matlab",
    "detected_script_languages": [
      "matlab"
    ],
    "script_groups": {
      "matlab": [
        "workspace/scripts/matlab/q1_baseline.m",
        "workspace/scripts/matlab/q1_main.m"
      ]
    },
    "next_skill": "matlab-code-reviewer",
    "handoff_context": {
      "method_plan": "workspace/problem/method_plan.json",
      "expected_results_dir": "workspace/results/",
      "expected_figures_dir": "workspace/figures/",
      "runtime_notes": [
        "beita-tianyuan-compatible",
        "avoid-heavy-toolboxes"
      ]
    }
  }
}
```

## Example 2: Route Python scripts to Python reviewer

Input state:

- `implementation.target = python`
- scripts exist under `workspace/scripts/python/`

Output:

```json
{
  "code_review_route": {
    "implementation_target": "python",
    "detected_script_languages": [
      "python"
    ],
    "script_groups": {
      "python": [
        "workspace/scripts/python/q2_baseline.py",
        "workspace/scripts/python/q2_main.py"
      ]
    },
    "next_skill": "python-code-reviewer",
    "handoff_context": {
      "method_plan": "workspace/problem/method_plan.json",
      "expected_results_dir": "workspace/results/",
      "expected_figures_dir": "workspace/figures/"
    }
  }
}
```

## Example 3: Blocked by target-script conflict

Input state:

- `implementation.target = matlab`
- only Python scripts are found

Output:

```json
{
  "code_review_route": {
    "status": "blocked",
    "blocked_items": [
      "implementation.target is matlab, but only Python scripts were found."
    ],
    "recommended_next_skill": "model-code-generator",
    "recommended_next_action": "Regenerate MATLAB scripts or revise the method plan if Python is intended."
  }
}
```

## Example 4: Mixed-language workflow

Input state:

- method plan explicitly allows mixed-language workflow
- Python preprocessing script and MATLAB model script both exist

Output:

```json
{
  "code_review_route": {
    "implementation_target": "mixed",
    "detected_script_languages": [
      "python",
      "matlab"
    ],
    "script_groups": {
      "python": [
        "workspace/scripts/python/preprocess.py"
      ],
      "matlab": [
        "workspace/scripts/matlab/q1_main.m"
      ]
    },
    "next_skill": [
      "python-code-reviewer",
      "matlab-code-reviewer"
    ],
    "handoff_context": {
      "method_plan": "workspace/problem/method_plan.json",
      "expected_results_dir": "workspace/results/",
      "expected_figures_dir": "workspace/figures/"
    }
  }
}
```

---
name: model-code-generator
description: Route validated method plans to Python or MATLAB code generation skills based on implementation target.
license: MIT
---

# Purpose

Route code generation to the correct language-specific implementation skill.

This skill reads a validated method plan, checks the `implementation.target` field, and hands off the task to either `python-model-code-generator` or `matlab-model-code-generator`.

This skill does not generate language-specific code itself. It only decides where code generation should happen.

# When to use

Use this skill:

- After `method-selector` has produced a validated method plan.
- After `data-auditor-cleaner` has confirmed that required data is ready or ready with documented warnings.
- When the workflow reaches the code generation stage.
- When the implementation target may be `python` or `matlab`.
- When a contest requires MATLAB / 北太天元 and the method plan should route to MATLAB-compatible code generation.

# Preconditions

The following should already exist or be provided:

- A validated problem parse.
- A validated problem classification artifact.
- A validated method plan.
- An `implementation` object in the method plan.
- Cleaned data or a clear statement that no tabular data is required.
- Data audit report and field mapping if data is used.

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

Supported targets:

- `python`
- `matlab`

Treat MATLAB / 北太天元 as the `matlab` target with compatibility notes.

# Inputs

Use or request:

- `workspace/problem/method_plan.json`, if available.
- `workspace/results/data_report.md`, if available.
- cleaned data paths under `workspace/data_clean/`
- expected result paths under `workspace/results/`
- expected figure paths under `workspace/figures/`
- implementation target and runtime notes
- user-provided contest requirements if they affect language choice

# Workflow

1. Read the method plan.
   - Confirm that it is validated.
   - Confirm that it includes `implementation.target`.
   - Confirm that the target is either `python` or `matlab`.
2. Check data readiness.
   - Confirm that the required cleaned data paths exist or are specified.
   - If cleaned data is required but missing, hand back to `data-auditor-cleaner`.
3. Resolve the implementation target.
   - If `implementation.target = python`, route to `python-model-code-generator`.
   - If `implementation.target = matlab`, route to `matlab-model-code-generator`.
   - If the contest requires MATLAB / 北太天元, prefer `matlab`.
   - If the target is missing or ambiguous, stop and report a blocker.
4. Prepare handoff context.
   - Include method plan path.
   - Include cleaned data paths.
   - Include expected outputs.
   - Include expected figures.
   - Include runtime notes.
   - Include any contest language constraints.
5. Hand off to the language-specific generator.
   - Do not generate code inside this router.
   - Do not change the method plan.
   - Do not invent data fields, labels, metrics, parameters, or outputs.

# Outputs

Produce a routing decision containing:

- `implementation_target`
- `runtime_notes`
- `next_skill`
- `handoff_context`
- `blocked_items`, if routing cannot proceed
- `recommended_next_action`, if blocked

# Output format

Prefer this JSON-compatible structure:

```json
{
  "code_generation_route": {
    "implementation_target": "matlab",
    "runtime_notes": [
      "beita-tianyuan-compatible",
      "avoid-heavy-toolboxes",
      "prefer-basic-matrix-and-table-operations"
    ],
    "next_skill": "matlab-model-code-generator",
    "handoff_context": {
      "method_plan": "workspace/problem/method_plan.json",
      "cleaned_data": [
        "workspace/data_clean/clean_data.csv"
      ],
      "expected_results_dir": "workspace/results/",
      "expected_figures_dir": "workspace/figures/",
      "expected_scripts_dir": "workspace/scripts/matlab/"
    }
  }
}
```

If blocked, use:

```json
{
  "code_generation_route": {
    "status": "blocked",
    "blocked_items": [
      "implementation.target is missing from the method plan."
    ],
    "recommended_next_skill": "method-selector",
    "recommended_next_action": "Add implementation.target with value python or matlab."
  }
}
```

# Routing rules

- `implementation.target = python` routes to `python-model-code-generator`.
- `implementation.target = matlab` routes to `matlab-model-code-generator`.
- MATLAB / 北太天元 compatibility should be expressed through `runtime_notes`, not a separate target.
- Do not route to both language generators unless the method plan explicitly requests a mixed-language workflow.
- Do not infer Python only because the agent can write Python easily.
- Do not infer MATLAB only because the problem is mathematical.
- If contest rules specify MATLAB / 北太天元, route to `matlab`.
- If the method plan and user request conflict, stop and ask for correction through `method-selector` or `workflow-orchestrator`.

# Target conventions

## Python

Use the Python branch when:

- the method plan says `implementation.target = python`
- the user explicitly requests Python
- the contest allows Python
- Python is used for data preprocessing, prototyping, or modeling

Expected script location:

```text
workspace/scripts/python/
```

Expected portable outputs:

```text
workspace/results/
workspace/figures/
```

## MATLAB / 北太天元

Use the MATLAB branch when:

- the method plan says `implementation.target = matlab`
- the contest requires MATLAB / 北太天元
- the user requests MATLAB-compatible code
- the team needs `.m` scripts for official computation

Expected script location:

```text
workspace/scripts/matlab/
```

Recommended runtime notes:

```text
beita-tianyuan-compatible
avoid-heavy-toolboxes
prefer-basic-matrix-and-table-operations
avoid-live-scripts
avoid-app-designer
save-portable-artifacts
```

Expected portable outputs:

```text
workspace/results/
workspace/figures/
```

# Rules

- Do not generate language-specific code.
- Do not review code.
- Do not modify the method plan.
- Do not select or change the modeling method.
- Do not fabricate data, fields, labels, parameters, metrics, outputs, or results.
- Do not write paper text.
- Do not bypass `data-auditor-cleaner` when cleaned data is required.
- Do not send code generation directly to `code-reviewer`.
- Do not route to a language-specific generator unless the implementation target is clear.
- Preserve all workflow gates.

# Verification

Before handoff, verify:

- The method plan exists or is provided.
- `implementation.target` is present.
- The target is either `python` or `matlab`.
- Runtime notes are preserved.
- Cleaned data requirements are satisfied or explicitly not needed.
- Expected script directory is language-specific:
  - `workspace/scripts/python/`
  - `workspace/scripts/matlab/`
- Expected result and figure directories remain language-neutral:
  - `workspace/results/`
  - `workspace/figures/`
- The next skill is correct.

# Failure modes

Stop and report a blocker if:

- The method plan is missing.
- The method plan is not validated.
- `implementation.target` is missing.
- `implementation.target` is not `python` or `matlab`.
- The user request conflicts with the method plan target.
- Cleaned data is required but unavailable.
- Required data fields are missing.
- Routing would require changing the modeling method.
- Routing would require inventing data or outputs.

# Stop conditions

This skill must stop instead of guessing when:

- The implementation target is ambiguous.
- Contest rules require a specific language but the method plan specifies another.
- The selected target cannot support the method with available artifacts.
- A mixed-language workflow is requested but not specified clearly.
- Proceeding would skip data readiness checks.

When stopping, output:

- blocker
- why it matters
- affected method plan item
- missing or conflicting information
- recommended repair skill
- next action

# Handoff

If `implementation.target = python`, hand off to:

```
python-model-code-generator
```

Include:

- method plan
- cleaned data paths
- field mapping
- expected result paths
- expected figure paths
- runtime notes
- implementation constraints

If `implementation.target = matlab`, hand off to:

```
matlab-model-code-generator
```

Include:

- method plan
- cleaned data paths
- field mapping
- expected result paths
- expected figure paths
- runtime notes
- MATLAB / 北太天元 compatibility requirements

If blocked, hand back to:

- `method-selector` for missing or wrong implementation target
- `data-auditor-cleaner` for missing cleaned data
- `workflow-orchestrator` for unresolved routing conflicts

# Examples

## Example 1: Route to MATLAB / 北太天元

Input state:

- method plan exists
- cleaned data exists
- `implementation.target = matlab`
- runtime notes include `beita-tianyuan-compatible`

Output:

```json
{
  "code_generation_route": {
    "implementation_target": "matlab",
    "runtime_notes": [
      "beita-tianyuan-compatible",
      "avoid-heavy-toolboxes"
    ],
    "next_skill": "matlab-model-code-generator",
    "handoff_context": {
      "method_plan": "workspace/problem/method_plan.json",
      "cleaned_data": [
        "workspace/data_clean/clean_data.csv"
      ],
      "expected_scripts_dir": "workspace/scripts/matlab/",
      "expected_results_dir": "workspace/results/",
      "expected_figures_dir": "workspace/figures/"
    }
  }
}
```

## Example 2: Route to Python

Input state:

- method plan exists
- cleaned data exists
- `implementation.target = python`

Output:

```json
{
  "code_generation_route": {
    "implementation_target": "python",
    "runtime_notes": [],
    "next_skill": "python-model-code-generator",
    "handoff_context": {
      "method_plan": "workspace/problem/method_plan.json",
      "cleaned_data": [
        "workspace/data_clean/clean_data.csv"
      ],
      "expected_scripts_dir": "workspace/scripts/python/",
      "expected_results_dir": "workspace/results/",
      "expected_figures_dir": "workspace/figures/"
    }
  }
}
```

## Example 3: Blocked by missing target

Input state:

- method plan exists
- no `implementation.target`

Output:

```json
{
  "code_generation_route": {
    "status": "blocked",
    "blocked_items": [
      "implementation.target is missing from the method plan."
    ],
    "recommended_next_skill": "method-selector",
    "recommended_next_action": "Update the method plan with implementation.target = python or matlab."
  }
}
```

## Example 4: Blocked by target conflict

Input state:

- contest requires MATLAB / 北太天元
- method plan says `implementation.target = python`

Output:

~~~json
{
  "code_generation_route": {
    "status": "blocked",
    "blocked_items": [
      "Contest requirements specify MATLAB / 北太天元, but the method plan targets Python."
    ],
    "recommended_next_skill": "method-selector",
    "recommended_next_action": "Revise the implementation target or explicitly justify the mixed-language workflow."
  }
}
写完后同步：

```bash
cp .claude/skills/model-code-generator/SKILL.md .codex/skills/model-code-generator/SKILL.md
diff -u .claude/skills/model-code-generator/SKILL.md .codex/skills/model-code-generator/SKILL.md
~~~

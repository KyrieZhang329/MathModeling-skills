---
name: matlab-model-code-generator
description: Generate executable MATLAB or Beita Tianyuan compatible modeling code from a validated method plan and cleaned data.
license: MIT
---

# Purpose

Generate MATLAB / 北太天元 compatible modeling code from a validated candidate method pool and cleaned data.

This skill implements the approved candidate methods (baseline, main candidates, optional improved) as `.m` scripts. It should produce code that reads cleaned data, runs the planned models, saves results in the `experiments/roundN/` output structure, generates a `run_summary.json`, and produces portable artifacts for downstream experiment reports, robustness checks, and paper writing.

This skill does not choose the model, clean raw data, review code, run final QA, or write paper sections.

# When to use

Use this skill:

- After `model-code-analyzer` hands off to `matlab-model-code-generator`.
- When `implementation.target = matlab`.
- When the contest requires MATLAB / 北太天元 compatible scripts.
- When the candidate method pool and cleaned data are ready.
- When executable `.m` scripts are needed for one or more candidate methods in a specific experiment round.

# Preconditions

The following should already exist or be provided:

- A validated problem parse.
- A validated problem classification artifact.
- A candidate method pool at `methods/Qx/qx_method_candidates.md`.
- `implementation.target = matlab`.
- Cleaned data under `workspace/data/data_clean/`, unless the model does not need tabular data.
- Data audit report and field mapping if data is used.
- `code/model-code-analyzer.md` — the code thinking document specifying round number, methods to implement, and output structure.
- Runtime notes if MATLAB / 北太天元 compatibility is required.

If the candidate method pool is missing, hand back to `method-selector`.

If `code/model-code-analyzer.md` is missing, hand back to `model-code-analyzer`.

If cleaned data is required but missing, hand back to `data-auditor-cleaner`.

If `implementation.target` is not `matlab`, hand back to `model-code-analyzer`.

# Inputs

Use or request:

- `methods/Qx/qx_method_candidates.md` — candidate method pool for each subquestion.
- `code/model-code-analyzer.md` — code thinking document.
- `workspace/data/data_report.md`, if available.
- Cleaned data under `workspace/data/data_clean/`.
- Field mapping from the data audit stage.
- Required model inputs per candidate method.
- Expected result outputs per candidate method.
- Expected figure outputs per candidate method.
- The round number (N for `roundN`).
- Runtime notes such as:
  - `beita-tianyuan-compatible`
  - `avoid-heavy-toolboxes`
  - `prefer-basic-matrix-and-table-operations`
  - `avoid-live-scripts`
  - `avoid-app-designer`
  - `save-portable-artifacts`

# Workflow

1. Read the code thinking document and candidate method pool.
   - Identify the target subquestion(s) and round number.
   - Identify which methods are to be implemented in this round.
   - Identify required inputs, expected outputs, validation needs, and robustness needs per method.
   - Do not change the selected methods or modeling route.

2. Confirm MATLAB target.
   - Confirm `implementation.target = matlab`.
   - If 北太天元 compatibility is required, use conservative MATLAB-compatible syntax.
   - Preserve all runtime notes.

3. Check data readiness.
   - Use cleaned data from `workspace/data/data_clean/`.
   - Do not read from `workspace/data/data_raw/` unless only inspecting metadata is explicitly requested.
   - Do not overwrite raw data.
   - Confirm required fields match the data audit report.

4. Plan script structure per subquestion.
   - Save MATLAB code under `code/matlab/Qx/` (e.g., `code/matlab/Q1/`).
   - One script per candidate method: `qx_m1_baseline.m`, `qx_m2_main.m`, `qx_m3_improved.m`.
   - A `README.md` in each code folder: run order, inputs, outputs, dependencies.
   - A `run_all.m` only if batch execution is useful.
   - Prefer plain `.m` scripts over Live Scripts.
   - Use functions only when they make the code clearer; if using functions, ensure function names match file names.

5. Generate code for each candidate method.
   - **Input**: Read from `workspace/data/data_clean/` using `readtable`, `readmatrix`, or `load`.
   - **Processing**: Implement the mathematical logic from the candidate method pool.
   - **Save results**: Save to `results/Qx/experiments/roundN/tables/` using `writetable`, `writematrix`, or `save`.
   - **Save figures**: Save to `results/Qx/experiments/roundN/figures/` using `saveas` or `print`.
   - **Save metrics**: Save to `results/Qx/experiments/roundN/metrics/`.
   - **Write log**: Use `diary` to write execution log to `results/Qx/experiments/roundN/logs/`.
   - **Write run_summary.json**: At end of `run_all.m` or the last script, generate `results/Qx/experiments/roundN/run_summary.json`.

6. Implement baseline code first.
   - Implement the baseline method first.
   - Save baseline outputs separately (e.g., `m1_scores.csv`).
   - Make baseline outputs comparable with other methods.

7. Implement main and optional methods.
   - Implement each candidate method with the same output conventions.
   - Use variable names that match the candidate method specification where practical.
   - Save intermediate values needed for explanation or robustness checks.

8. Implement run_summary.json generation.
   - After all methods have executed, write `run_summary.json` using MATLAB's `jsonencode`:
     ```matlab
     run_summary = struct();
     run_summary.question = 'Q1';
     run_summary.round = 'round1';
     run_summary.execution_timestamp = datestr(now, 'yyyy-mm-ddTHH:MM:SS');
     run_summary.implementation_target = 'matlab';
     run_summary.random_seed = 2026;
     run_summary.methods = struct(...
         'method_id', 'M1', ...
         'method_name', 'equal_weight_scoring', ...
         'script', 'code/matlab/Q1/q1_m1_baseline.m', ...
         'status', 'success');
     % ... add more methods and details
     f = fopen('results/Q1/experiments/round1/run_summary.json', 'w');
     fprintf(f, '%s', jsonencode(run_summary));
     fclose(f);
     ```

9. Add a README.md per code folder.
   - State the subquestion and round.
   - List scripts and their roles.
   - State run order.
   - List expected input data files.
   - List expected output paths.
   - State MATLAB / 北太天元 compatibility notes.
   - State known portability risks.

10. Hand off to `code-reviewer`.
    - The router should route `.m` scripts to `matlab-code-reviewer`.

# Outputs

Produce MATLAB implementation artifacts such as:

- `code/matlab/Q1/README.md`
- `code/matlab/Q1/q1_m1_baseline.m`
- `code/matlab/Q1/q1_m2_main.m`
- `code/matlab/Q1/q1_m3_improved.m` (if applicable)
- `code/matlab/Q1/run_all.m` (if useful)
- `results/Q1/experiments/roundN/` (created by running the scripts):
  - `tables/*.csv`
  - `figures/*.png`
  - `metrics/*.json`
  - `logs/*.log`
  - `run_summary.json`

# Output format

Prefer this JSON-compatible summary:

```json
{
  "matlab_code_generation_summary": {
    "implementation_target": "matlab",
    "subquestion": "Q1",
    "round": "round1",
    "runtime_notes": ["beita-tianyuan-compatible", "avoid-heavy-toolboxes"],
    "generated_scripts": [
      "code/matlab/Q1/q1_m1_baseline.m",
      "code/matlab/Q1/q1_m2_main.m",
      "code/matlab/Q1/run_all.m"
    ],
    "data_inputs": [
      "workspace/data/data_clean/indicator_data.csv"
    ],
    "result_outputs": [
      "results/Q1/experiments/round1/tables/m1_scores.csv",
      "results/Q1/experiments/round1/tables/m2_scores.csv"
    ],
    "figure_outputs": [
      "results/Q1/experiments/round1/figures/m1_score_bar.png"
    ],
    "run_instructions": [
      "Run code/matlab/Q1/q1_m1_baseline.m",
      "Run code/matlab/Q1/q1_m2_main.m"
    ],
    "known_risks": [
      "Compatibility with 北太天元 should be checked if toolbox-specific functions are introduced."
    ],
    "recommended_next_skill": "code-reviewer"
  }
}
```

# MATLAB / 北太天元 coding rules

Use conservative MATLAB-compatible code.

Prefer:
- plain `.m` scripts
- basic matrix operations
- `readtable`, `writetable`
- `readmatrix`, `writematrix`
- `save`, `load`
- `plot`, `bar`, `scatter`, `histogram`
- `rng(seed)`
- clear relative paths using `fullfile`
- explicit result saving to `results/Qx/experiments/roundN/`
- `jsonencode` for `run_summary.json` (or manual JSON string construction for older MATLAB)
- simple loops and vectorized operations when readable

Avoid unless explicitly approved:
- Live Scripts
- App Designer
- GUI code
- Simulink dependencies
- parallel toolbox
- deep learning toolbox
- symbolic toolbox
- heavy toolbox-specific functions
- version-specific MATLAB features not supported by 北太天元
- third-party packages
- absolute local paths
- scripts that only display results without saving them

For 北太天元 compatibility:
- Prefer simple MATLAB syntax.
- Avoid relying on advanced toolbox functions.
- Save portable artifacts (`.csv`, `.mat`, `.png`).
- If uncertain whether a function is supported, choose a simpler implementation.
- Keep comments clear enough for manual translation or debugging.

# Path rules

Use relative paths where practical.

Recommended paths:
```text
workspace/data/data_clean/
code/matlab/Qx/
results/Qx/experiments/roundN/
  ├── tables/
  ├── figures/
  ├── metrics/
  ├── logs/
  └── run_summary.json
```

Example path setup:
```matlab
dataDir = fullfile('workspace', 'data_clean');
resultDir = fullfile('results', 'Q1', 'experiments', 'round1');
tableDir = fullfile(resultDir, 'tables');
figureDir = fullfile(resultDir, 'figures');
metricDir = fullfile(resultDir, 'metrics');
logDir = fullfile(resultDir, 'logs');

if ~exist(tableDir, 'dir'), mkdir(tableDir); end
if ~exist(figureDir, 'dir'), mkdir(figureDir); end
if ~exist(metricDir, 'dir'), mkdir(metricDir); end
if ~exist(logDir, 'dir'), mkdir(logDir); end
```

# Artifact rules

- Read cleaned data from `workspace/data/data_clean/`.
- Do not modify files under `workspace/data/data_raw/`.
- Save MATLAB scripts under `code/matlab/Qx/`.
- Save numeric outputs under `results/Qx/experiments/roundN/tables/`.
- Save figures under `results/Qx/experiments/roundN/figures/`.
- Save metrics under `results/Qx/experiments/roundN/metrics/`.
- Save logs under `results/Qx/experiments/roundN/logs/`.
- Generate `run_summary.json` under `results/Qx/experiments/roundN/`.
- Use clear file names with method prefix.
- Separate baseline outputs from main model outputs.
- Save important intermediate results as `.mat` files when useful.

# Randomness rules

If randomness is used:
- Set a fixed seed with `rng(seed)`.
- Record the seed in comments and `run_summary.json`.
- Avoid hidden random initialization.

```matlab
rng(2026);
```

# Commenting rules

Use comments to explain:
- data inputs and their source paths
- key variables and their meanings
- model steps and non-obvious formulas
- output files and their destinations
- assumptions inherited from the method plan
- random seed and reproducibility notes

Avoid over-commenting trivial syntax.

# Rules

- Do not select or change the model — implement what the candidate method pool specifies.
- Do not clean raw data.
- Do not fabricate data, labels, parameters, metrics, or results.
- Do not write paper text.
- Do not perform final QA.
- Do not depend on heavy MATLAB toolboxes unless explicitly approved.
- Do not hide assumptions inside code.
- Do not silently drop rows or columns.
- Do not overwrite raw data.
- Always generate `run_summary.json`.
- Always create the full `experiments/roundN/` directory structure.
- Do not claim the code is reviewed or correct before `matlab-code-reviewer`.
- Separate baseline outputs from main method outputs.

# Verification

Before handing off, verify:

- `implementation.target = matlab`.
- Every generated script maps to a method in the candidate method pool.
- Baseline code exists for every subquestion unless explicitly impossible.
- Scripts are saved under `code/matlab/Qx/`.
- Scripts write outputs to `results/Qx/experiments/roundN/`.
- Cleaned data paths are used.
- No raw data file is overwritten.
- Random seeds are fixed where needed.
- `run_summary.json` generation is implemented.
- Figures are saved to `results/Qx/experiments/roundN/figures/`.
- MATLAB / 北太天元 compatibility risks are listed.
- A `README.md` exists in the code folder.
- The next skill is `code-reviewer`.

# Failure modes

Stop and report a blocker if:

- The candidate method pool is missing.
- `code/model-code-analyzer.md` is missing.
- `implementation.target` is not `matlab`.
- Cleaned data is required but unavailable.
- Required data fields are missing.
- The selected method cannot be implemented without unsupported toolbox functions.
- The expected output is not defined.
- Generating code would require inventing data, labels, parameters, or metrics.
- MATLAB / 北太天元 compatibility requirements conflict with the selected method.

# Stop conditions

This skill must stop instead of guessing when:

- Model variables cannot be mapped to cleaned data fields.
- Important parameters are unspecified and cannot be safely defaulted.
- The method requires a toolbox or function that may be unavailable under 北太天元 and no simpler alternative is specified.
- The code would need to overwrite raw data.
- A script would produce results that cannot be traced to the candidate method pool.
- Continuing would change the mathematical meaning of the model.

When stopping, output:
- blocker
- why it matters
- affected subquestion or model
- missing information or artifact
- safe partial implementation if any
- recommended next skill
- next action

# Handoff

After generating MATLAB scripts, hand off to:

`code-reviewer`

The handoff should include:
- generated `.m` script paths
- code folder `README.md` path
- candidate method pool path
- cleaned data paths
- field mapping
- expected result paths under `results/Qx/experiments/roundN/`
- expected figure paths
- runtime notes
- MATLAB / 北太天元 compatibility requirements
- run instructions
- known risks and assumptions

`code-reviewer` should route the scripts to `matlab-code-reviewer`.

Do not hand off directly to `robustness-checker` or `result-report-generator`.

# Examples

## Example 1: Generate MATLAB evaluation model code for round 1

Input state:
- Q1 candidate pool: M1 (equal-weight baseline), M2 (entropy-TOPSIS).
- Cleaned indicator data exists.
- Round: round1.
- `implementation.target = matlab`.
- Runtime notes include `beita-tianyuan-compatible`.

Output:
```json
{
  "matlab_code_generation_summary": {
    "implementation_target": "matlab",
    "subquestion": "Q1",
    "round": "round1",
    "generated_scripts": [
      "code/matlab/Q1/q1_m1_baseline.m",
      "code/matlab/Q1/q1_m2_entropy_topsis.m",
      "code/matlab/Q1/run_all.m",
      "code/matlab/Q1/README.md"
    ],
    "data_inputs": [
      "workspace/data/data_clean/indicator_data.csv"
    ],
    "result_outputs": [
      "results/Q1/experiments/round1/tables/m1_equal_weight_scores.csv",
      "results/Q1/experiments/round1/tables/m2_entropy_topsis_scores.csv",
      "results/Q1/experiments/round1/run_summary.json"
    ],
    "figure_outputs": [
      "results/Q1/experiments/round1/figures/m1_score_bar.png",
      "results/Q1/experiments/round1/figures/m2_weight_bar.png"
    ],
    "run_instructions": [
      "Run code/matlab/Q1/q1_m1_baseline.m",
      "Run code/matlab/Q1/q1_m2_entropy_topsis.m"
    ],
    "known_risks": [
      "Indicator direction must match the method plan before interpretation.",
      "jsonencode requires MATLAB R2016b+; for older/北太天元, manual JSON string construction is used."
    ],
    "recommended_next_skill": "code-reviewer"
  }
}
```

## Example 2: Blocked by missing candidate pool

```json
{
  "blocked_items": [
    "The candidate method pool is missing at methods/Q1/q1_method_candidates.md."
  ],
  "recommended_next_skill": "method-selector",
  "recommended_next_action": "Generate the candidate method pool for Q1 before MATLAB code generation."
}
```

## Example 3: Blocked by heavy toolbox dependency

```json
{
  "blocked_items": [
    "The selected implementation appears to require a toolbox-specific optimization function, which conflicts with the 北太天元 compatibility constraint."
  ],
  "affected_model": "Q3 M2",
  "recommended_next_skill": "method-selector",
  "recommended_next_action": "Revise the candidate method pool to use a basic matrix-based or explicitly supported solver approach."
}
```

# MATLAB / 北太天元 Guidelines

This document defines conservative MATLAB / 北太天元 compatibility rules for the `matlab` implementation target.

The goal is to generate and review `.m` scripts that are likely to run in a contest environment and produce portable artifacts for downstream analysis and paper writing.

## Target definition

Use:

```json
{
  "implementation": {
    "target": "matlab",
    "runtime_notes": [
      "beita-tianyuan-compatible"
    ]
  }
}
```

Do not create a separate `beita-tianyuan` target.

北太天元 should be handled as a MATLAB-compatible runtime requirement under the `matlab` target.

## Basic principle

Write conservative MATLAB-compatible code.

Prefer simple, readable, contest-friendly `.m` scripts over advanced language features, toolboxes, GUI tools, or environment-specific workflows.

When uncertain whether a function is supported in 北太天元, choose a simpler implementation.

## Recommended runtime notes

For MATLAB / 北太天元 workflows, use runtime notes such as:

```text
beita-tianyuan-compatible
avoid-heavy-toolboxes
prefer-basic-matrix-and-table-operations
avoid-live-scripts
avoid-app-designer
save-portable-artifacts
```

These notes should be preserved by:

```text
model-code-generator
matlab-model-code-generator
code-reviewer
matlab-code-reviewer
```

## File style

Prefer:

- plain `.m` scripts
- small helper functions only when useful
- one script per subquestion when tasks are separable
- clear execution order
- optional `run_all.m` when it improves reproducibility

Avoid:

- Live Scripts
- App Designer
- GUI code
- Simulink dependencies
- undocumented helper functions
- scripts that depend on manual workspace variables
- scripts that only work after running commands interactively

## Script location

Save MATLAB scripts under:

```text
workspace/scripts/matlab/
```

Examples:

```text
workspace/scripts/matlab/q1_baseline.m
workspace/scripts/matlab/q1_main.m
workspace/scripts/matlab/q1_improved.m
workspace/scripts/matlab/run_all.m
```

## Data location

Read cleaned data from:

```text
workspace/data_clean/
```

Do not modify:

```text
workspace/data_raw/
```

The raw data directory should be treated as read-only.

If data cleaning is required, it should be handled by `data-auditor-cleaner` before MATLAB model code generation.

## Output location

Save result artifacts under:

```text
workspace/results/
```

Save figure artifacts under:

```text
workspace/figures/
```

Recommended result formats:

```text
csv
mat
xlsx
```

Recommended figure formats:

```text
png
pdf
fig
```

Use `png` as the default paper-ready format.

Use `.mat` when saving intermediate arrays, model variables, or data structures for later MATLAB review.

## Path rules

Use relative paths where practical.

Recommended setup:

```matlab
dataDir = fullfile('workspace', 'data_clean');
resultDir = fullfile('workspace', 'results');
figureDir = fullfile('workspace', 'figures');

if ~exist(resultDir, 'dir')
    mkdir(resultDir);
end

if ~exist(figureDir, 'dir')
    mkdir(figureDir);
end
```

Avoid hard-coded local paths:

```text
/Users/...
C:\Users\...
D:\...
```

A script with local absolute paths is not portable and should be fixed during code review.

## Data I/O

Prefer common MATLAB-compatible data I/O functions:

```text
readtable
writetable
readmatrix
writematrix
load
save
```

Use `readtable` / `writetable` for tabular data when field names matter.

Use `readmatrix` / `writematrix` for purely numeric arrays.

Use `load` / `save` for `.mat` artifacts.

If a compatibility issue is suspected with a newer function, choose a simpler or more widely supported alternative.

## Figure saving

Figures should be saved explicitly.

Example:

```matlab
figure;
plot(x, y);
xlabel('x');
ylabel('y');
title('Q1 result');
grid on;
saveas(gcf, fullfile(figureDir, 'q1_result.png'));
close;
```

Do not rely only on visual display.

A figure used in the paper must exist as a file under:

```text
workspace/figures/
```

## Randomness

If randomness is used, set a fixed seed.

Example:

```matlab
rng(2026);
```

Use fixed seeds for:

- Monte Carlo simulation
- random sampling
- random initialization
- stochastic optimization
- train-test split if implemented in MATLAB

Record the seed in comments or summary outputs.

## Matrix and vector rules

MATLAB uses 1-based indexing.

Check:

- row vs column vectors
- matrix dimensions
- vector transposes
- element-wise operations
- implicit expansion assumptions
- division by zero
- `NaN` and `Inf`
- weight vector length
- constraint matrix size

Use element-wise operators when intended:

```matlab
.*
./
.^
```

Do not accidentally use matrix operations when element-wise operations are required.

## Table and field rules

When using tables:

- keep field names aligned with the data audit report
- do not silently rename important columns
- do not silently drop rows or columns
- preserve units and indicator directions
- document missing-value handling

If column names are not valid MATLAB identifiers, handle them explicitly and document the transformation.

## Toolbox policy

Avoid heavy toolbox dependencies unless explicitly approved in the method plan.

Avoid by default:

- App Designer
- GUI code
- Simulink
- parallel toolbox
- deep learning toolbox
- symbolic toolbox
- specialized optimization toolbox functions
- version-specific MATLAB features
- third-party packages

Allowed by default:

- basic matrix operations
- basic statistics implemented manually if needed
- basic plotting
- table / matrix I/O
- simple optimization logic implemented directly when practical

If a method requires a toolbox-specific function, the skill should either:

1. use a simpler compatible alternative, or
2. stop and return to `method-selector` for method revision or explicit approval.

## Optimization code

For optimization problems, prefer transparent implementations when contest scale allows it.

Acceptable approaches include:

- direct formula computation
- enumeration for small discrete cases
- greedy baseline
- manually implemented dynamic programming
- manually implemented heuristic search
- basic linear or integer programming only if supported and approved

Avoid hiding the entire solution inside a toolbox call if the function may not exist in 北太天元.

The paper should be able to explain the algorithm, not only the function call.

## Baseline and main model separation

Keep baseline and main model outputs separate.

Recommended naming:

```text
q1_baseline.m
q1_main.m
q1_improved.m

q1_baseline_results.csv
q1_main_results.csv
q1_improved_results.csv
```

Do not overwrite baseline results with main model results.

Do not mix baseline and main model outputs in a way that makes comparison impossible.

## Comments

Use comments to explain:

- input files
- output files
- model steps
- non-obvious formulas
- parameter choices
- inherited assumptions
- compatibility notes

Avoid excessive comments for trivial syntax.

## Minimum script header

A useful MATLAB script should start with a short header like:

```matlab
% Q1 main model
% Input: workspace/data_clean/indicator_data.csv
% Output:
%   workspace/results/q1_main_results.csv
%   workspace/figures/q1_ranking.png
% Notes:
%   MATLAB / 北太天元 compatible style.
%   Avoids heavy toolbox dependencies.

clear; clc; close all;
```

This makes the script easier to inspect under contest time pressure.

## Review checklist

Before trusting MATLAB / 北太天元 code, check:

- script is a plain `.m` file
- script can run from the project root
- script maps to the method plan
- baseline and main model are separated
- cleaned data is used
- raw data is not overwritten
- paths are relative
- table fields match the data report
- matrix dimensions are correct
- row and column vectors are consistent
- randomness is controlled
- results are saved under `workspace/results/`
- figures are saved under `workspace/figures/`
- heavy toolbox dependencies are avoided or approved
- compatibility risks are listed

## Common blockers

Stop and return to the appropriate upstream skill when:

- the method plan is missing
- `implementation.target` is not `matlab`
- cleaned data is missing
- required fields are missing
- code requires unsupported toolbox functions
- compatibility requirements conflict with the selected method
- the script cannot produce required result artifacts
- the script would need to overwrite raw data
- fixing the code would change the mathematical model

## Practical rule

Use the simplest MATLAB-compatible implementation that can produce the required outputs and survive review.

For contest work, a clear `.m` script with saved artifacts is usually better than a clever solution that depends on fragile environment-specific features.


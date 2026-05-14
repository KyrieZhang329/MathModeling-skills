---
name: symbol-table-builder
description: Build and maintain a unified global symbol table for all subquestions, ensuring consistent notation across the entire mathematical modeling solution.
license: MIT
---

# Purpose

Build and maintain a single unified symbol table for the entire mathematical modeling contest solution.

This skill reads the problem parse and all subquestion candidate method pools, extracts every variable, parameter, set, index, function, and output symbol, checks for naming conflicts across subquestions, and produces a single consistent symbol table that all downstream skills (method explanations, code, paper sections) must reference.

The goal: ensure that the same concept uses the same symbol everywhere, and different concepts never share a symbol.

This skill does not define model equations, select methods, or write paper sections.

# When to use

Use this skill:

- After `problem-parser` and `problem-classifier` have produced validated artifacts.
- After `method-selector` has produced the first round of candidate method pools.
- BEFORE code generation and paper writing — symbols must be fixed early.
- When the user says: "build the symbol table", "unify the notation", "check for symbol conflicts", "define all variables".
- When a new subquestion's methods introduce new symbols that need to be integrated.

# Preconditions

The following should already exist or be provided:

- A validated problem parse with preliminary variables and relationships.
- Problem classification artifacts.
- Candidate method pools for all subquestions (or at least those that have been planned).
- Question dependency map (if available).

# Inputs

Use or request:

- `workspace/problem/problem-parser/problem_parse.json` — for preliminary variables.
- `methods/Qx/qx_method_candidates.md` — for method-specific variables.
- The question dependency map (to know which subquestions share variables).
- Any existing notation the team has already used.

# Workflow

1. Extract all variables from the problem parse.
   - Observable quantities.
   - Controllable quantities.
   - External parameters.
   - Unknowns to estimate.
   - Sets and indices implied by the problem context.

2. Extract all variables from each subquestion's candidate method pool.
   - Decision variables.
   - State variables.
   - Parameters.
   - Intermediate computation variables.
   - Output variables.
   - Indices and sets.

3. Build the initial symbol assignment.
   - Assign a unique symbol to each distinct concept.
   - Prefer standard mathematical notation conventions:
     - $i, j, k$ for indices
     - $n, m$ for counts
     - $x$ for raw/input values, $y$ for output/target
     - $w, \omega$ for weights
     - $S$ for scores, $R$ for rankings
     - $C$ for cost, $P$ for probability
     - Greek letters for parameters ($\alpha, \beta, \lambda, \theta$)
     - Capital letters for sets ($I, J, K$)
     - Bold for vectors ($\mathbf{x}$), capital bold for matrices ($\mathbf{X}$)

4. Detect and resolve conflicts.
   - If two different concepts have the same symbol → reassign one.
   - If the same concept has different symbols in different subquestions → unify to one symbol.
   - If a subquestion's notation conflicts with standard convention → flag for discussion.

5. Identify cross-subquestion shared symbols.
   - Mark symbols that appear in multiple subquestions.
   - Ensure shared symbols have consistent definitions across subquestions.
   - If Q1's output becomes Q2's input, the handoff variable must have a single consistent symbol.

6. Categorize every symbol.
   - **Sets and Indices**: $i \in I$, $j \in J$, $t \in T$
   - **Parameters (given)**: known constants from data or problem statement
   - **Parameters (estimated)**: parameters determined by the model
   - **Decision Variables**: what the model chooses/controls
   - **State Variables**: computed from decisions and parameters
   - **Input Variables**: raw data values
   - **Output Variables**: final results delivered to the paper
   - **Intermediate Variables**: computation-only, not in paper

7. Produce the global symbol table.
   - Save as `planning/symbol_table.md`.
   - Include: symbol, meaning, type, unit (if applicable), subquestions using it, first defined in.

8. Hand off to downstream skills.
   - All downstream skills must reference this table.
   - `final-method-explainer` and `paper-section-writer` should verify their notation against it.

# Outputs

Produce exactly one document:

- `planning/symbol_table.md`

# Output format

```markdown
# Global Symbol Table

> Last updated: [timestamp]
> Based on: problem parse + candidate method pools for Q1, Q2, Q3, Q4

## Sets and Indices

| Symbol | Meaning | Domain / Range | Used In |
|--------|---------|---------------|---------|
| $i$ | City index | $i = 1, 2, \ldots, m$ | Q1, Q2 |
| $j$ | Indicator index | $j = 1, 2, \ldots, n$ | Q1 |
| $t$ | Time period index | $t = 1, 2, \ldots, T$ | Q2, Q3 |
| $k$ | Resource type index | $k = 1, 2, \ldots, K$ | Q3 |

## Parameters (Given)

| Symbol | Meaning | Value / Source | Unit | Used In |
|--------|---------|---------------|------|---------|
| $x_{ij}$ | Raw value of indicator $j$ for city $i$ | From cleaned indicator matrix | varies | Q1 |
| $C_k$ | Total capacity of resource $k$ | From problem statement | units of resource | Q3 |
| $D_{it}$ | Demand at city $i$ in period $t$ | From cleaned demand data | units of demand | Q2, Q3 |

## Parameters (Estimated by Model)

| Symbol | Meaning | Estimation Method | Used In |
|--------|---------|-------------------|---------|
| $\omega_j$ | Entropy weight of indicator $j$ | $\omega_j = \frac{1 - e_j}{\sum_{k=1}^n (1 - e_k)}$ | Q1 |
| $\hat{D}_{it}$ | Predicted demand at city $i$ in period $t$ | ARIMA model output | Q2 |

## Decision Variables

| Symbol | Meaning | Domain | Unit | Used In |
|--------|---------|--------|------|---------|
| $a_{ik}$ | Amount of resource $k$ allocated to city $i$ | $a_{ik} \geq 0$ | units of resource | Q3 |
| $z_{ik}$ | Binary: 1 if city $i$ receives resource $k$ | $z_{ik} \in \{0, 1\}$ | — | Q3 |

## State Variables (Computed)

| Symbol | Meaning | Computation | Used In |
|--------|---------|------------|---------|
| $S_i$ | Composite score of city $i$ | $S_i = \sum_{j=1}^n \omega_j \cdot x'_{ij}$ | Q1 |
| $C_i$ | Relative closeness (TOPSIS) | $C_i = \frac{d_i^-}{d_i^+ + d_i^-}$ | Q1 |
| $R_i$ | Rank of city $i$ | Sorted by $C_i$ descending | Q1 |

## Output Variables

| Symbol | Meaning | Format | Used In |
|--------|---------|--------|---------|
| $\mathbf{R}$ | Final ranking vector | $\mathbf{R} = (R_1, R_2, \ldots, R_m)$ | Q1 |
| $\hat{\mathbf{D}}$ | Demand forecast matrix | $\hat{\mathbf{D}} \in \mathbb{R}^{m \times T}$ | Q2 |
| $\mathbf{A}^*$ | Optimal allocation matrix | $\mathbf{A}^* \in \mathbb{R}^{m \times K}$ | Q3 |

## Cross-Subquestion Shared Symbols

| Symbol | Meaning | Q1 | Q2 | Q3 | Q4 | Consistency Check |
|--------|---------|----|----|----|----|------------------|
| $i$ | City index | $1..m$ | $1..m$ | $1..m$ | — | ✅ consistent |
| $S_i$ | City score | Output | Input (feature) | — | — | ✅ Q1→Q2 handoff defined |
| $\hat{D}_{it}$ | Predicted demand | — | Output | Input (parameter) | — | ✅ Q2→Q3 handoff defined |

## Symbol Conflict Resolution Log

| Date | Conflict | Resolution |
|------|----------|------------|
| 2026-05-14 | Q1 used $w_j$ for weight, Q2 used $w_t$ for window size | Q1: $\omega_j$ (weight), Q2: $w_t$ (window size) |
| 2026-05-14 | Q2 used $x_t$ for time series, Q1 used $x_{ij}$ for indicator values | Keep $x$ for raw data across all Q; distinguish by indices |

## Notation Conventions

- **Indices**: lowercase Latin letters ($i$, $j$, $k$, $t$)
- **Parameters**: lowercase Greek or Latin with descriptive subscripts
- **Decision variables**: lowercase Latin ($a$, $z$, $y$)
- **Vectors**: bold lowercase ($\mathbf{x}$, $\mathbf{w}$)
- **Matrices**: bold uppercase ($\mathbf{X}$, $\mathbf{A}$)
- **Sets**: uppercase italic ($I$, $J$, $T$)
- **Estimated values**: hat notation ($\hat{x}$, $\hat{D}$)
- **Optimal values**: star superscript ($x^*$, $\mathbf{A}^*$)
- **Normalized values**: prime notation ($x'$)

## References

- Problem parse: `workspace/problem/problem-parser/problem_parse.json`
- Q1 candidate pool: `methods/Q1/q1_method_candidates.md`
- Q2 candidate pool: `methods/Q2/q2_method_candidates.md`
- ...
```

# Rules

- Build the symbol table early — before code generation and paper writing.
- Same concept = same symbol everywhere.
- Different concepts = different symbols always.
- Every symbol must have: name, meaning, type, domain/unit, and which subquestions use it.
- Distinguish variable types: given parameter, estimated parameter, decision variable, state variable, input, output.
- Follow standard mathematical notation conventions.
- Document all conflict resolutions.
- Update the table when new methods introduce new symbols.
- All downstream skills should verify their symbols against this table.

# Verification

Before handing off, verify:

- Every variable from the problem parse is represented.
- Every variable from each subquestion's candidate pool is represented.
- No two distinct concepts share the same symbol.
- No single concept has different symbols in different subquestions.
- Each symbol has a type category.
- Cross-subquestion shared symbols are identified.
- Notation conventions are documented.
- Conflict resolution log records all changes.

# Failure modes

Stop and report a blocker if:

- The problem parse is missing.
- Candidate method pools are missing for all subquestions.
- A fundamental notational ambiguity makes it impossible to assign consistent symbols.
- Two subquestions define the same symbol with incompatible meanings and neither can be changed without breaking the method.

# Stop conditions

This skill must stop instead of guessing when:

- Not enough subquestions have been planned to build a useful table.
- A symbol conflict cannot be resolved without the modeler's input.
- The problem context is too vague to determine appropriate notation.

When stopping, output:
- the blocker
- what symbols can still be defined
- what input is needed from the user
- recommended next action

# Handoff

After producing `planning/symbol_table.md`, hand off to:

`workflow-orchestrator`
— which will route downstream skills to use this table as a reference.

The handoff should include:
- The symbol table path.
- A summary of any unresolved conflicts or decisions needed from the modeler.

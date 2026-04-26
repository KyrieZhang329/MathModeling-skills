---
name: data-auditor-cleaner
description: Audit, clean, summarize, and prepare contest data for modeling.
license: MIT
---

# Purpose

Audit, clean, summarize, and prepare mathematical modeling contest data for downstream modeling.

This skill converts raw contest data and method data requirements into cleaned, traceable modeling data. It identifies data fields, checks data quality, handles missing values and anomalies with explicit rules, generates basic exploratory summaries, and produces artifacts that downstream code and paper sections can trust.

This skill does not select models, implement the main model, run final experiments, perform robustness analysis, or write paper sections.

# When to use

Use this skill:

- After `method-selector` has produced a validated method plan.
- When raw data files are available under `workspace/data_raw/`.
- When data fields, units, missing values, abnormal values, or encodings need to be audited.
- Before `model-code-generator`.
- Before writing paper claims based on data.
- When the team needs a reproducible data cleaning record.

# Preconditions

The following should already exist or be provided:

- A validated problem parse.
- A validated problem classification artifact.
- A validated method plan.
- Raw data files or a clear statement that no raw data is provided.
- Required data fields or expected inputs from `method-selector`.

If the method plan is missing, hand back to `method-selector`.

If raw data is unavailable but required, stop and report the missing data instead of fabricating it.

# Inputs

Use or request:

- `workspace/problem/problem_parse.json`, if available.
- `workspace/problem/problem_classification.json`, if available.
- `workspace/problem/method_plan.json`, if available.
- Raw data under `workspace/data_raw/`.
- Data dictionaries, attachment descriptions, column descriptions, or unit notes.
- User-provided explanation of fields if the dataset lacks metadata.
- Contest rules about external data, if applicable.

# Workflow

1. Preserve raw data.
   - Treat `workspace/data_raw/` as read-only.
   - Do not overwrite, edit, or silently transform raw source files.
   - Record the original file names and formats.

2. Inventory the available data.
   - List every dataset.
   - Identify file format, sheet names if applicable, row count, column count, and apparent encoding.
   - Map available fields to the method plan's required data fields.
   - Mark missing required fields explicitly.

3. Audit field definitions and units.
   - Identify each column's likely meaning.
   - Check data type, unit, range, category meaning, and time granularity.
   - Flag unclear, mixed, inconsistent, or undocumented fields.

4. Audit data quality.
   - Check missing values.
   - Check duplicate rows or duplicate keys.
   - Check abnormal values and outliers.
   - Check impossible values, such as negative counts or invalid dates.
   - Check category inconsistencies, spelling variants, and encoding issues.
   - Check time continuity if the data is temporal.
   - Check whether sample size is sufficient for the selected methods.

5. Propose cleaning operations.
   - Explain each operation before applying it.
   - Separate safe cleaning from assumption-based cleaning.
   - Keep a record of changed, removed, imputed, or transformed values.
   - Do not silently delete data.

6. Produce cleaned data.
   - Save cleaned data under `workspace/data_clean/`.
   - Keep file names traceable to the raw sources.
   - Save reproducible cleaning logic under `workspace/scripts/` if code is generated.
   - Save a data audit report under `workspace/results/`.

7. Generate exploratory summaries.
   - Provide descriptive statistics for key variables.
   - Provide simple EDA figures only when they support field understanding or later modeling.
   - Do not create decorative figures.

8. Evaluate data readiness.
   - State whether the cleaned data can support the method plan.
   - Identify remaining data risks.
   - Recommend whether to proceed to `model-code-generator` or return to `method-selector`.

# Outputs

Produce data preparation artifacts such as:

- `workspace/results/data_report.md`
- `workspace/data_clean/clean_data.csv` or equivalent cleaned files
- `workspace/scripts/data_cleaning.py` or `workspace/scripts/data_cleaning.m`, if code is needed
- `workspace/figures/eda_*.png` or equivalent EDA figures, if useful
- field mapping between raw data and method requirements
- list of unresolved data risks
- recommended next skill

# Output format

Prefer this JSON-compatible report summary:

```json
{
  "data_audit_summary": {
    "raw_files": [
      {
        "path": "workspace/data_raw/example.csv",
        "format": "csv",
        "rows": 100,
        "columns": 8,
        "status": "readable"
      }
    ],
    "method_data_coverage": [
      {
        "required_field": "demand",
        "matched_field": "daily_demand",
        "status": "matched",
        "risk": "unit must be confirmed"
      }
    ],
    "overall_readiness": "ready_with_warnings"
  },
  "field_audit": [
    {
      "field": "daily_demand",
      "type": "numeric",
      "unit": "unknown",
      "missing_count": 0,
      "outlier_risk": "medium",
      "notes": "Unit should be confirmed before final interpretation."
    }
  ],
  "cleaning_actions": [
    {
      "action": "standardize date format",
      "target": "date",
      "reason": "mixed date formats detected",
      "risk": "low"
    }
  ],
  "generated_artifacts": [
    "workspace/results/data_report.md",
    "workspace/data_clean/clean_data.csv",
    "workspace/scripts/data_cleaning.py"
  ],
  "remaining_risks": [
    "Unit for daily_demand is not explicitly stated."
  ],
  "recommended_next_skill": "model-code-generator"
}
```

If a JSON block is too rigid for the situation, use a concise Markdown report with the same fields.

# Data audit checklist

Check at least the following:

## File-level checks

- file path
- file format
- readability
- encoding
- sheet names for spreadsheets
- row count
- column count
- duplicate files
- missing referenced attachments

## Field-level checks

- column names
- likely meaning
- data type
- unit
- value range
- valid categories
- missing values
- duplicate keys
- abnormal values
- impossible values
- mixed formats
- inconsistent labels
- relationship to method requirements

## Time-related checks

Use when time fields exist:

- date parsing
- time granularity
- time continuity
- missing periods
- duplicated timestamps
- irregular intervals
- possible seasonality
- chronological leakage risk

## Modeling-readiness checks

- whether required fields are present
- whether sample size supports the method plan
- whether labels exist for supervised learning tasks
- whether target variables are clearly defined
- whether features are available before prediction time
- whether units match the model equations
- whether external data use is allowed and documented

# Cleaning rules

- Preserve raw data unchanged.
- Prefer reversible transformations.
- Record every cleaning decision.
- Do not delete rows or columns without stating why.
- Do not impute missing values without stating the method and risk.
- Do not normalize or standardize variables unless downstream methods require it.
- Do not merge datasets without checking keys, units, and row counts.
- Do not treat correlation as causation in the data report.
- Do not create features that leak future information into prediction tasks.
- Do not hide data insufficiency.

# Rules

- Do not select or change the main modeling route unless data limitations make the current method plan infeasible.
- Do not generate final model code.
- Do not run final experiments.
- Do not write paper sections.
- Do not fabricate missing data.
- Do not silently fix ambiguous fields.
- Do not overwrite raw data.
- Do not produce decorative figures.
- Do not claim the data is ready if required fields are missing.
- Mark high-risk cleaning assumptions explicitly.
- If the method plan cannot be supported by the data, recommend returning to `method-selector`.

# Verification

Before handing off, verify:

- Raw data remains unchanged.
- All raw files are inventoried.
- Required fields from the method plan are mapped or marked missing.
- Missing values, duplicates, outliers, units, and type issues are checked.
- Cleaning actions are documented.
- Cleaned data is saved under `workspace/data_clean/` or a save path is recommended.
- Data report is saved under `workspace/results/` or a save path is recommended.
- EDA figures, if generated, support modeling decisions.
- Remaining data risks are listed.
- The next skill is either `model-code-generator` or `method-selector`, depending on readiness.

# Failure modes

Stop and report a blocker if:

- Raw data is required but unavailable.
- A referenced attachment is missing.
- The data file is unreadable or corrupted.
- Required fields cannot be identified.
- Units or field meanings are too ambiguous for safe modeling.
- The method plan requires labels or target variables that do not exist.
- Cleaning would require inventing values or contest data.
- External data is required but contest rules do not permit or clarify it.

# Stop conditions

This skill must stop instead of guessing when:

- A cleaning decision would materially change the meaning of the data.
- Missing data prevents the selected method from being feasible.
- Field names are too ambiguous to map to model variables.
- The user requests modeling results before data readiness is established.
- Continuing would require fabricating data, units, labels, or target variables.

When stopping, output:

- the blocker
- why it matters
- affected datasets and fields
- safe partial audit results if any
- missing information needed
- recommended next action

# Handoff

If data is ready or ready with documented warnings, hand off to:

`model-code-generator`

The handoff should include:

- cleaned data path
- data report path
- field mapping
- required model inputs
- remaining data risks
- EDA figure paths if available
- cleaning script path if available

If data is not sufficient for the selected method plan, hand off to:

`method-selector`

The handoff should include:

- unsupported data requirements
- missing fields
- infeasible model assumptions
- recommended constraints for revised method selection

# Examples

## Example 1: Data ready with warnings

Input state:

- Method plan requires `date`, `sales`, and `price`.
- Raw CSV contains all three fields.
- `date` uses mixed formats.
- `sales` has two missing values.

Output:

```json
{
  "data_audit_summary": {
    "overall_readiness": "ready_with_warnings",
    "reason": "All required fields are present, but date format and sales missing values require documented cleaning."
  },
  "cleaning_actions": [
    {
      "action": "standardize date format",
      "target": "date",
      "reason": "mixed date formats detected",
      "risk": "low"
    },
    {
      "action": "flag missing sales values",
      "target": "sales",
      "reason": "missing values may affect prediction",
      "risk": "medium"
    }
  ],
  "generated_artifacts": [
    "workspace/data_clean/clean_data.csv",
    "workspace/results/data_report.md"
  ],
  "recommended_next_skill": "model-code-generator"
}
```

## Example 2: Missing required field

Input state:

- Method plan requires demand data.
- Raw file contains price and inventory but no demand field.

Output:

```json
{
  "data_audit_summary": {
    "overall_readiness": "blocked"
  },
  "blocked_items": [
    "The method plan requires demand data, but no demand field is available."
  ],
  "recommended_next_skill": "method-selector",
  "recommended_next_action": "Revise the method plan to avoid direct demand prediction or provide a valid demand data source."
}
```

## Example 3: Raw data protection

Input state:

- User asks to clean the original file directly inside `workspace/data_raw/`.

Output:

```json
{
  "blocked_items": [
    "Raw data under workspace/data_raw/ must remain unchanged."
  ],
  "recommended_next_action": "Create cleaned copies under workspace/data_clean/ and keep a reproducible cleaning record."
}
```

# QA Checklist

This document provides a final checklist before assembling or submitting a mathematical modeling contest paper.

QA should find blockers. It should not hide them.

## Final decision

Use three possible QA states:

- `passed`: final assembly is allowed
- `passed_with_warnings`: final assembly is allowed, but limitations should remain visible
- `failed`: final assembly is blocked

If any required subquestion is unanswered, QA fails.

## Problem coverage

Check:

- every original subquestion is listed
- every subquestion has a corresponding answer
- every answer has a model, result, or justified non-model response
- dependencies between subquestions are handled
- final conclusions map back to the original problem

Blocking issues:

- missing subquestion
- wrong interpretation of a subquestion
- conclusion does not answer the question asked

## Method consistency

Check:

- task type matches the chosen method
- method plan matches paper description
- baseline exists for every main model unless justified
- improved model is clearly marked as optional if not essential
- rejected methods are not later described as used

Blocking issues:

- paper describes a different method than the code implements
- main model has no baseline and no justification
- method cannot produce the required output

## Data consistency

Check:

- raw data remains unchanged
- cleaned data is documented
- field meanings and units are clear
- missing values and outliers are handled or discussed
- data used in scripts matches the data report
- derived features are explained

Blocking issues:

- required data field missing
- raw data overwritten
- undocumented cleaning changes results materially
- unit mismatch affects interpretation

## Code and result consistency

Check:

- scripts exist for generated results
- run instructions are available
- output files are saved under `workspace/results/`
- figures are saved under `workspace/figures/`
- paper numbers match result files
- random seeds are fixed when needed

Blocking issues:

- result in paper cannot be found in output files
- code output contradicts paper text
- script cannot be mapped to any subquestion
- runtime failure blocks required result

## Baseline and robustness

Check:

- baseline result exists where needed
- main model is compared with baseline
- robustness or sensitivity checks exist
- fragile conclusions are marked
- robustness claims do not exceed completed checks

Blocking issues:

- claim of improvement without baseline
- claim of stability without robustness check
- planned robustness check presented as completed

## Figures and tables

Check:

- every referenced figure exists or is clearly marked as planned
- every referenced table exists
- every figure or table has a source artifact
- captions explain the takeaway
- visual evidence supports the related claim
- no decorative figure is used as evidence

Blocking issues:

- figure or table referenced but missing
- table contains invented values
- figure contradicts result file
- visual overstates the evidence

## Paper writing

Check:

- assumptions are necessary
- notation is defined before use
- equations match variables and units
- numerical claims have artifacts
- limitations are included
- conclusion does not introduce new claims
- abstract does not overstate results

Blocking issues:

- unsupported numerical claim
- invented reference or result
- hidden limitation that affects the conclusion
- conclusion exceeds evidence

## Artifact traceability

For each major claim, identify at least one supporting artifact:

| Claim type             | Expected artifact                    |
| ---------------------- | ------------------------------------ |
| problem interpretation | problem parse                        |
| method choice          | method plan                          |
| data statement         | data report                          |
| numerical result       | result file                          |
| figure claim           | figure source file                   |
| robustness claim       | robustness report                    |
| code-based result      | reviewed script and output           |
| final conclusion       | paper section plus supporting result |

If the artifact cannot be found, the claim should be removed or marked incomplete.

## Repair routing

Use the earliest skill that can fix the issue.

| Issue                            | Repair skill            |
| -------------------------------- | ----------------------- |
| problem misunderstood            | `problem-parser`        |
| wrong task type                  | `problem-classifier`    |
| method does not fit task or data | `method-selector`       |
| data issue                       | `data-auditor-cleaner`  |
| missing result generation        | `model-code-generator`  |
| code or output mismatch          | `code-reviewer`         |
| missing robustness evidence      | `robustness-checker`    |
| unsupported visual               | `figure-table-planner`  |
| unsupported or unclear writing   | `paper-section-writer`  |
| multiple blockers across stages  | `workflow-orchestrator` |

## Minimal QA report

A QA report should include:

```json
{
  "qa_status": "failed",
  "final_assembly_allowed": false,
  "blocking_issues": [
    {
      "issue": "Q3 has no validated result artifact.",
      "why_it_matters": "The final paper cannot answer all required subquestions.",
      "repair_skill": "model-code-generator"
    }
  ],
  "minor_issues": [
    {
      "issue": "Caption for Fig.2 does not state the main takeaway.",
      "repair_skill": "paper-section-writer"
    }
  ],
  "unsupported_claims": [
    {
      "claim": "The model is globally stable.",
      "reason": "Only limited sensitivity checks were completed."
    }
  ],
  "recommended_next_skill": "workflow-orchestrator"
}
```

## **Practical rule**

If QA fails, do not assemble the final paper.

Fix the earliest blocker first.


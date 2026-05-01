# Modeling Workflow

[English](./modeling-workflow.md) | [简体中文](./modeling-workflow-zh.md)

This document explains how the skills in `MathModeling-skills` are connected during a mathematical modeling contest.

The goal is not to force every contest problem into the same solution pattern. The goal is to reduce common workflow mistakes: reading the problem too loosely, choosing methods too early, writing code before the model is clear, writing paper conclusions before results exist, or writing claims that cannot be traced back to result files.

## Workflow overview

```text
workflow-orchestrator
→ problem-parser
→ problem-classifier
→ related-paper-analyzer
→ method-selector
→ data-auditor-cleaner
→ model-code-generator (router)
   ├── python-model-code-generator
   └── matlab-model-code-generator
→ code-reviewer (router)
   ├── python-code-reviewer
   └── matlab-code-reviewer
→ robustness-checker
→ figure-table-planner
→ paper-section-writer
→ quality-assurance-auditor
→ workflow-orchestrator
```

Each stage should leave an artifact that the next stage can inspect.

## **Stage-by-stage workflow**

### **1. workflow-orchestrator**

**Role:** Check the current project state and decide the next skill.

**Inputs:**

- current workspace tree
- existing artifacts
- user status update
- contest deadline or constraints

**Outputs:**

- current stage
- missing artifacts
- blockers
- next skill
- next actions

**Common blockers:**

- no problem statement
- inconsistent workspace state
- user asks to skip a required gate

**Next step:** Usually `problem-parser`.

### **2. problem-parser**

**Role:** Read the problem and turn it into a structured task description.

**Inputs:**

- problem statement
- attachments or data descriptions
- contest rules related to data usage or submission format

**Outputs:**

- background
- main goal
- objects
- subquestions
- constraints
- data inventory
- required outputs
- ambiguities and risks

**Common blockers:**

- unreadable problem statement
- missing attachment
- unclear subquestions

**Next step:** `problem-classifier`.

### **3. problem-classifier**

**Role:** Classify each subquestion by task type.

**Inputs:**

- parsed subquestions
- required outputs
- data inventory
- constraints

**Outputs:**

- primary type for each subquestion
- secondary type if needed
- classification reason
- candidate method families
- classification risks

**Common blockers:**

- missing problem parse
- unclear required output
- subquestion too vague to classify

**Next step:** `related-paper-analyzer`.

### **4. related-paper-analyzer**

**Role:** Collect and analyze relevant papers, reports, and transferable method ideas before final method selection.

**Inputs:**

- problem parse
- problem classification
- contest constraints
- known data scope
- accessible references if available

**Outputs:**

- literature summary
- candidate transferable methods
- reference risks
- assumptions worth checking
- notes for method comparison

**Common blockers:**

- no accessible references
- references do not match the task type
- source quality is unclear
- user expects invented citations or unsupported copying

**Next step:** `method-selector`.

### **5. method-selector**

**Role:** Compare 2-4 feasible modeling schemes for each subquestion and recommend one execution route.

**Inputs:**

- problem parse
- problem classification
- related paper analysis
- data inventory
- contest constraints
- team constraints if relevant

**Outputs:**

- 2-4 candidate schemes per subquestion
- recommended scheme for execution
- baseline model
- main model
- optional improved model
- rejected methods and reasons
- expected inputs and outputs
- validation and robustness plan

**Common blockers:**

- data does not support the intended model
- required output is unknown
- no meaningful baseline can be defined

**Next step:** Usually `data-auditor-cleaner`.

### **6. data-auditor-cleaner**

**Role:** Check and prepare data for modeling.

**Inputs:**

- raw data
- method plan
- field descriptions
- data dictionaries if available

**Outputs:**

- data report
- cleaned data
- field mapping
- cleaning script if needed
- unresolved data risks

**Common blockers:**

- missing raw data
- unreadable file
- required field missing
- ambiguous unit or field meaning

**Next step:** `model-code-generator` if data is ready; otherwise return to `method-selector`.

### **7. model-code-generator**

**Role:** Route the validated method plan to the correct code generation branch.

**Inputs:**

- method plan
- cleaned data
- expected outputs
- validation needs
- implementation target

**Outputs:**

- routed implementation target
- model scripts
- result output paths
- figure output paths
- run instructions
- implementation notes

**Common blockers:**

- no validated method plan
- cleaned data missing
- required fields unavailable
- method cannot be implemented with available data

**Next step:** `code-reviewer`.

### **8. code-reviewer**

**Role:** Route reviewed script families to the correct language-specific reviewer before relying on their outputs.

**Inputs:**

- model scripts
- cleaned data
- method plan
- run instructions
- error logs if any
- implementation target

**Outputs:**

- reviewed or corrected scripts
- fixed issue list
- remaining risks
- verified output paths

**Common blockers:**

- script missing
- runtime error requires changing the model
- code does not match method plan
- output cannot be linked to a subquestion

**Next step:** `robustness-checker`.

### **9. robustness-checker**

**Role:** Check whether results are stable enough to support claims.

**Inputs:**

- reviewed code
- baseline results
- main model results
- method plan
- data report

**Outputs:**

- robustness report
- sensitivity tables
- error metrics
- baseline comparison
- stable and fragile conclusions

**Common blockers:**

- missing baseline
- missing model results
- undefined claim or metric
- check would require invented data

**Next step:** `figure-table-planner`.

### **10. figure-table-planner**

**Role:** Decide which figures and tables are needed in the paper, and what argument each one should support.

**Inputs:**

- model results
- robustness results
- method plan
- existing figures
- expected paper structure

**Outputs:**

- planned figures
- planned tables
- source artifacts
- section mapping
- caption requirements
- visual risks

**Common blockers:**

- requested figure has no source artifact
- result table missing
- visual would overstate the evidence

**Next step:** `paper-section-writer`.

### **11. paper-section-writer**

**Role:** Draft paper sections from existing artifacts.

**Inputs:**

- problem parse
- method plan
- data report
- model results
- robustness report
- figure-table plan

**Outputs:**

- paper section drafts
- artifact mapping
- unsupported claims list
- incomplete sections

**Common blockers:**

- result missing
- figure-table plan missing
- requested claim unsupported
- conclusion would exceed evidence

**Next step:** `quality-assurance-auditor`.

### **12. quality-assurance-auditor**

**Role:** Perform the final solution audit before assembly or submission.

**Inputs:**

- all paper sections
- problem parse
- method plan
- data report
- scripts
- results
- figures
- robustness report

**Outputs:**

- QA status
- blocking issues
- minor issues
- repair plan
- final assembly decision

**Common blockers:**

- unanswered subquestion
- unsupported numerical claim
- missing robustness evidence
- figure referenced but not available
- model, code, and paper disagree

**Next step:** Back to `workflow-orchestrator`.

## **Workflow gates**

These gates are intentional:

- No method selection before problem parsing.
- No code generation before method planning is complete.
- No paper claims before result artifacts exist.
- No model superiority claim without baseline and robustness evidence.
- No final assembly before QA passes.
- Raw data must remain unchanged.

If a gate blocks progress, report the blocker and route back to the correct upstream skill.

## **When to go backward**

Going backward is normal. Use the earliest skill that can fix the issue.

| **Problem**                                 | **Go back to**          |
| ------------------------------------------- | ----------------------- |
| The problem was misunderstood               | `problem-parser`        |
| The task type is wrong                      | `problem-classifier`    |
| The data cannot support the selected method | `method-selector`       |
| Raw or cleaned data has issues              | `data-auditor-cleaner`  |
| Code fails or does not match the method     | `code-reviewer`         |
| Conclusions are unstable                    | `robustness-checker`    |
| A figure or table has no evidence           | `figure-table-planner`  |
| A paper claim is unsupported                | `paper-section-writer`  |
| Final submission has blockers               | `workflow-orchestrator` |

## **Artifact map**

Suggested workspace layout:

```text
workspace/
├── problem/
├── data_raw/
├── data_clean/
├── scripts/
├── results/
├── figures/
├── paper_sections/
└── final_paper/
```

Typical contents:

| **Path**                    | **Contents**                                                 |
| --------------------------- | ------------------------------------------------------------ |
| `workspace/problem/`        | problem statement, parsed problem, classification, method plan |
| `workspace/data_raw/`       | original data files; should remain unchanged                 |
| `workspace/data_clean/`     | cleaned data and derived features                            |
| `workspace/scripts/`        | generated Python or MATLAB scripts                           |
| `workspace/results/`        | result tables, metrics, reports, robustness outputs          |
| `workspace/figures/`        | generated figures used or planned for the paper              |
| `workspace/paper_sections/` | section drafts                                               |
| `workspace/final_paper/`    | assembled final paper and submission materials               |

## **Minimal run example**

A new contest task might start like this:

```text
Use workflow-orchestrator. The problem is in workspace/problem/problem.pdf and the raw data is in workspace/data_raw/. Start from problem-parser and do not skip stages.
```

A typical sequence is:

1. Parse the problem.
2. Classify each subquestion.
3. Select baseline, main, and optional improved models.
4. Audit and clean data.
5. Generate model code.
6. Review code and outputs.
7. Run robustness checks.
8. Plan figures and tables.
9. Draft paper sections.
10. Run final QA.

## **Notes**

This workflow does not replace modeling judgment. It is a guardrail.

You can skip or merge stages when there is a good reason, but the choice should be explicit. If a result, figure, or claim cannot be traced back to an artifact, treat it as unfinished.

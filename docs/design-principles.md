# Design Principles

This document records the design principles behind `MathModeling-skills`.

The repository is not meant to make an agent more confident. It is meant to make the workflow harder to misuse.

## Start from the problem, not the model

Do not start with model names.

Start with:

- what the problem asks for
- what objects are being studied
- what constraints exist
- what data is available
- what output is required
- what variables and relationships can be checked

A model is only useful if it produces the required output under the available data and constraints.

## Work by subquestion

Most contest problems are mixed.

Do not classify the whole problem by its title. Classify each subquestion separately.

A single contest problem may include:

- evaluation
- prediction
- optimization
- simulation
- mechanism modeling
- data analysis

Each subquestion should have its own task type, method choice, expected output, and validation plan.

## Keep a baseline

Every main model should have a baseline unless there is a clear reason not to.

A baseline is not decoration. It should be:

- simple
- runnable
- explainable
- comparable to the main model
- able to produce the same type of output

Without a baseline, claims like “better,” “more accurate,” or “more stable” are usually unsupported.

## Keep artifacts traceable

Important outputs should be traceable.

A paper claim should connect back to one or more of:

- problem parse
- method plan
- data report
- script
- result table
- figure
- robustness report
- paper section
- QA report

If a claim cannot be traced, treat it as unfinished.

## Do not invent missing evidence

If data, results, figures, references, or robustness checks are missing, report the gap.

Do not fill the gap with plausible-looking text.

This is especially important for:

- numerical results
- model performance claims
- rankings
- optimal decisions
- robustness conclusions
- citations
- figure descriptions

## Preserve raw data

Raw data should be treated as read-only.

Put original files in:

```text
workspace/data_raw/
```

Put cleaned or transformed files in:

```text
workspace/data_clean/
```

Every cleaning step should be documented, especially when it changes missing values, outliers, units, categories, or derived features.

## **Prefer simple methods first**

A complex method is only useful if it solves a real problem that a simpler method cannot.

Avoid complex methods when:

- the data is small
- labels are missing
- the method is hard to explain
- the team cannot debug it under time pressure
- validation would be weak
- the output does not match the problem requirement

Start simple, then improve only where needed.

## **Separate planning, coding, checking, and writing**

The workflow separates stages on purpose:

- problem parsing is not method selection
- method selection is not code generation
- code generation is not code review
- robustness checking is not paper writing
- QA is not final paper assembly

This separation prevents the paper from drifting away from the actual model and results.

## **Be honest about uncertainty**

Uncertainty is not a failure.

Mark:

- ambiguous problem wording
- missing data
- weak assumptions
- unstable results
- fragile conclusions
- untested scenarios

A limited but honest solution is better than an overclaimed one.

## **Human judgment remains necessary**

The skills provide guardrails. They do not replace modeling judgment.

Users still need to decide:

- whether an assumption is acceptable
- whether a method is appropriate
- whether a result is meaningful
- whether a conclusion should be stated strongly or cautiously


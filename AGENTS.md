# Core Philosophy

- Start from goals, objects, constraints, data, outputs, variables, relationships, and checkable conclusions.
- Do not start from model names or favorite techniques.
- Separate assumptions, observations, derivations, and validated conclusions.

# Workflow Discipline

- Parse the problem before classifying it.
- Classify the task before selecting methods.
- Generate a candidate method pool before committing to one method.
- Validate the method plan before writing code or paper text.
- Run multi-round experiments before locking in the final method.
- Keep every change minimal, traceable, and easy to review.

# Workflow Gates

- Do not select methods before the problem is parsed and classified.
- Do not generate code before a candidate method pool exists.
- Do not write numerical paper claims before results exist.
- Do not hand a subquestion to the paper writer before all three critical rules are satisfied.
- Do not assemble the final paper before QA passes.

# Per-Question Delivery Chain (MANDATORY)

For each subquestion Qx, it is FORBIDDEN to treat "code ran successfully" as "the subquestion is done."

Every subquestion must complete the following chain:

```
Stage 1:  建模手提出多个候选方法
          → methods/Qx/qx_method_candidates.md

Stage 2:  编程手实现多候选方法, 运行实验
          → code/Qx/*.py (or *.m)
          → results/Qx/experiments/roundN/

Stage 3:  编程手生成实验报告, 对比各方法
          → results/Qx/experiments/roundN/qx_experiment_report_roundN.md

Stage 4:  建模手根据实验报告修改、淘汰或合并方法
          → methods/Qx/qx_method_iteration_log.md

Stage 5:  多轮迭代后 (重复 Stage 2-4), 建模手确定最终方法
          → State: Final Method Selected (confirmed by modeler)

Stage 6:  AI/建模手写最终方法详解
          → methods/Qx/qx_final_method_explanation.md

Stage 7:  编程手/result-report-generator 生成最终结果分析
          → results/Qx/reports/qx_final_result_analysis.md

Stage 8:  robustness-checker 执行稳健性分析
          → robustness/Qx/qx_robustness_report.md

Stage 9:  solution-package-builder 生成论文手材料包
          → results/Qx/reports/qx_solution_package_for_writer.md

Stage 10: 论文手撰写 Qx 论文段落
          → paper/sections/qx.tex (or .md)

Stage 11: 建模手审核论文模型部分, 编程手审核论文结果和图表

Stage 12: 审核通过后 Qx 状态改为 Finalized
```

If `qx_final_method_explanation.md`, `qx_final_result_analysis.md`, or `qx_solution_package_for_writer.md` is missing, that subquestion MUST NOT enter the paper writing stage.

# Minimum Files Per Subquestion

Each subquestion Qx must have at least these 6 files before being considered "Ready for Writer":

| # | File | Primary Author | Purpose |
|---|------|---------------|---------|
| 1 | `methods/Qx/qx_method_candidates.md` | 建模手 + method-selector | 候选方法池 |
| 2 | `methods/Qx/qx_method_iteration_log.md` | 建模手 + 编程手 | 方法迭代记录 |
| 3 | `methods/Qx/qx_final_method_explanation.md` | 建模手 + final-method-explainer | 最终方法详解 |
| 4 | `results/Qx/reports/qx_experiment_report.md` | 编程手 + result-report-generator | 多方法实验对比 |
| 5 | `results/Qx/reports/qx_final_result_analysis.md` | 编程手 + result-report-generator | 最终结果分析 |
| 6 | `results/Qx/reports/qx_solution_package_for_writer.md` | solution-package-builder | 论文手材料包 |

# Three Critical Rules (ENFORCED AT GATES)

These three rules are enforced by `workflow-orchestrator` and `paper-section-writer`. Violation of any rule means the subquestion is NOT Ready for Writer.

## Rule 1: 没有最终方法详解，不准写最终论文

`paper-section-writer` MUST NOT write final paper sections for Qx based on `qx_method_candidates.md` or early method plans. The file `methods/Qx/qx_final_method_explanation.md` MUST exist first.

## Rule 2: 没有最终结果分析，不准交给论文手

Program execution outputs alone are NOT a deliverable. The programmer (or `result-report-generator`) MUST produce `results/Qx/reports/qx_final_result_analysis.md` before the writer can proceed.

## Rule 3: 论文手只看材料包，不从零散 results 里猜

The paper writer's primary source for Qx is `results/Qx/reports/qx_solution_package_for_writer.md`. If this file is missing, Qx is NOT Ready for Writer, and the writer should not hunt through scattered results.

# Figure Type Classification

All figures must be classified into one of four types:

| Type | Name | Audience | Paper Use |
|------|------|----------|-----------|
| Type 1 | 诊断图 Diagnostic | 建模手 (modeler) | NOT for paper — internal model debugging only |
| Type 2 | 对比图 Comparison | 建模手 + 论文手 | MAY appear in paper — supports method selection narrative |
| Type 3 | 论文图 Paper | 论文读者 (judges) | MUST appear in paper — supports main claims |
| Type 4 | 附录图 Appendix | 论文读者 (supplementary) | Referenced from main text, shown in appendix |

Type 1 diagnostic figures must NOT appear in the final paper. Type 3 paper figures must be publication-quality (≥300 dpi, clear labels, informative captions).

# Experiments Output Structure

Every round of experiments MUST output to:

```
results/Qx/experiments/roundN/
├── figures/          # All generated figures
├── tables/           # All result tables (.csv)
├── metrics/          # All evaluation metrics (.csv, .json)
├── logs/             # Execution logs
└── run_summary.json  # Machine-readable execution record
```

`run_summary.json` must record: question ID, round number, methods executed, their statuses, input/output files, metrics summaries, random seeds, and environment info.

# Project Structure (Recommended)

```
project/
├── planning/                   # Global planning: parse, classify, symbol table, assumptions, dashboard
│   ├── parse/                  # Problem parse outputs
│   ├── classification/         # Problem classification outputs
│   ├── symbol_table.md         # Unified symbol table across all subquestions
│   ├── model_assumptions.md    # Global model assumptions
│   ├── question_dependency.md  # Subquestion dependency map
│   └── progress_dashboard.md   # Live progress tracking
├── methods/                    # 建模手区: method design and iteration
│   └── Qx/                     # Per-subquestion method artifacts
├── code/                       # 编程手区: executable code
│   └── Qx/                     # Per-subquestion code
├── results/                    # 正式结果区: experiment outputs and reports
│   └── Qx/
│       ├── experiments/        # Raw experiment outputs by round
│       │   └── roundN/
│       └── reports/            # Analysis reports and solution packages
├── robustness/                 # 稳健性分析区
│   └── Qx/                     # Per-subquestion robustness artifacts
├── paper/                      # 论文手区: final paper
│   ├── sections/               # Paper section drafts
│   ├── figures/                # Final paper figures (Type 3 + Type 4)
│   ├── main.tex                # Main paper file
│   └── qa_report.md            # QA report
├── docs/                       # Long-term reference materials
│   ├── references/             # Method references
│   ├── past_papers/            # Example excellent papers
│   └── templates/              # Paper templates, report templates
└── scratch/                    # Temporary exploration (not for final delivery)
```

# Symbol Discipline

- Create a unified symbol table early (at `planning/symbol_table.md`).
- Same meaning must use the same symbol across all subquestions.
- Different meanings must use different symbols.
- Distinguish: decision variables, state variables, parameters, inputs, intermediate values, and outputs.
- Every symbol must be defined before first use.
- Include units where applicable.

# Model Assumptions

- State model assumptions at `planning/model_assumptions.md` early, and refine them.
- Distinguish "necessary" assumptions (model breaks without them) from "simplifying" assumptions (model works approximately without them).
- Every assumption must be linked to a modeling need.
- State the possible impact if an assumption is violated.
- Do not include filler assumptions that don't affect the model.

# Artifact Discipline

- Treat problem parses, method plans, cleaned data, scripts, results, figures, robustness reports, and paper sections as traceable artifacts.
- Claims must point back to artifacts.
- Preserve artifact lineage from problem statement to final delivery.

# Modeling Rules

- Define the problem structure before proposing methods.
- Generate 2-4 candidate methods per subquestion — do not commit to the first idea.
- Match model choice to task type, available data, interpretability needs, and contest constraints.
- Every main model must have a baseline.
- Do not choose complex models only because they look advanced.
- Do not invent missing evidence.
- Final method selection happens AFTER experiments, by the modeler reviewing experiment reports.

# Coding Rules

- Generate code only from a validated method plan.
- Code must produce output in the experiments/roundN/ structure.
- Prefer clear, minimal, reviewable implementations over clever ones.
- Do not hide assumptions inside code.
- Save all outputs to files (CSV, JSON, PNG) — do not rely on console output only.
- Use fixed random seeds for reproducibility.

# Paper Writing Rules

- Write from validated artifacts, not from guesswork.
- The solution package (`qx_solution_package_for_writer.md`) is the primary source.
- Keep claims aligned with actual models, data, figures, and checks.
- Do not fabricate references, numerical results, experiments, or conclusions.
- Do not write final paper sections for Qx if any of the three critical rules are violated.
- Method description must match the final method explanation, not an early candidate pool.
- Results must come from the final result analysis, not raw experiment outputs.
- Mention eliminated methods to demonstrate thoroughness.

# Verification and QA

- Check logic, units, definitions, and output consistency before handoff.
- Robustness or sensitivity checks are required before final delivery.
- Every major claim must have at least one supporting robustness check or a stated limitation.
- Final delivery requires QA approval.
- QA checks BOTH paper quality AND workflow completeness (all 6 minimum files per subquestion).
- Flag uncertainty explicitly instead of smoothing it over.
- Do not fabricate data.
- Do not hide uncertainty.
- Do not approve final assembly with blocking issues.

# Team Role Responsibilities

| Role | Primary Artifacts | Review Responsibility |
|------|------------------|----------------------|
| 建模手 (Modeler) | `methods/Qx/qx_method_candidates.md`, `methods/Qx/qx_method_iteration_log.md`, `methods/Qx/qx_final_method_explanation.md` | Review paper's model description |
| 编程手 (Programmer) | `code/Qx/*`, `results/Qx/experiments/`, `results/Qx/reports/qx_final_result_analysis.md` | Review paper's result claims and figures |
| 论文手 (Writer) | `paper/sections/qx.tex`, final assembly | N/A (reviewed by modeler + programmer) |
| AI/工作流 | All intermediate reports, solution package, figure plans, robustness reports | Automated consistency checks |

# Git Discipline

- Push only intentional changes.
- Do not push raw data, scratch files, intermediate experiment outputs, or generated figures unless they are final.
- Use `.gitignore` to exclude `scratch/`, `results/Qx/experiments/` (keep only `results/Qx/reports/`).
- Resolve merge conflicts carefully — do not discard others' work.

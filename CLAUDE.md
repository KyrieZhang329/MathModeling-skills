# Core Philosophy

- Start from goals, objects, constraints, data, outputs, variables, relationships, and checkable conclusions.
- Do not start from model names.
- Separate assumptions from validated conclusions at every stage.

# Workflow Discipline

- Follow the full workflow in order instead of skipping ahead.
- Keep file edits minimal and traceable.
- Do not overwrite validated artifacts casually.
- Only generate code from a validated method plan.

# Workflow Gates

- Do not select methods before the problem is parsed.
- Do not generate code before the method plan is validated.
- Do not write numerical paper claims before results exist.
- Do not hand a subquestion to the paper writer before all three critical rules are satisfied.
- Do not assemble the final paper before QA passes.

# Three Critical Rules

Rule 1 — 没有最终方法详解，不准写最终论文:
  `paper-section-writer` must not write final paper sections for Qx unless
  `methods/Qx/qx_final_method_explanation.md` exists.

Rule 2 — 没有最终结果分析，不准交给论文手:
  Raw experiment outputs are not a deliverable. `results/Qx/reports/qx_final_result_analysis.md`
  must exist before the writer can proceed for Qx.

Rule 3 — 论文手只看材料包，不从零散 results 里猜:
  The writer's primary source for Qx is `results/Qx/reports/qx_solution_package_for_writer.md`.
  If this file is missing, Qx is NOT Ready for Writer.

# Per-Question Delivery Chain

For each subquestion Qx, it is forbidden to treat "code ran successfully" as "the subquestion is done."

Every subquestion must go through:
```
1. method-selector        → methods/Qx/qx_method_candidates.md
2. model-code-analyzer    → code/model-code-analyzer.md (with experiments/roundN/ plan)
3. code generators        → code/Qx/*.py (or code/matlab/Qx/*.m)
4. result-report-generator → results/Qx/experiments/roundN/qx_experiment_report_roundN.md
5. [loop 2-4 until method converges]
6. final-method-explainer → methods/Qx/qx_final_method_explanation.md
7. result-report-generator → results/Qx/reports/qx_final_result_analysis.md
8. robustness-checker      → robustness/Qx/qx_robustness_report.md
9. figure-table-planner    → methods/Qx/qx_figure_table_plan.md
10. solution-package-builder → results/Qx/reports/qx_solution_package_for_writer.md
11. paper-section-writer   → paper/sections/qx.tex (or .md)
12. modeler review + programmer review
```

Minimum 6 files per subquestion before "Ready for Writer":
- methods/Qx/qx_method_candidates.md
- methods/Qx/qx_method_iteration_log.md
- methods/Qx/qx_final_method_explanation.md
- results/Qx/reports/qx_experiment_report.md
- results/Qx/reports/qx_final_result_analysis.md
- results/QX/reports/qx_solution_package_for_writer.md

# Full Skill List (24 skills)

| Skill | Role |
|-------|------|
| workflow-orchestrator | 总调度：跟踪每小问状态，决定下一步 |
| problem-parser | 题目解析：拆成目标/对象/约束/数据/输出/子问题 |
| problem-classifier | 题型分类：评价/预测/优化/机理/分类/图/仿真/混合 |
| related-paper-analyzer | 论文分析：从真实文献提炼方法，不编造引用 |
| method-selector | 候选方法池生成：每小问 2-4 个候选方案 |
| symbol-table-builder | 全局符号表构建：统一所有小问的符号约定 |
| model-assumptions-builder | 全局模型假设：区分必要/简化，评估影响 |
| data-auditor-cleaner | 数据审计清洗：副本操作，不碰原始数据 |
| model-code-analyzer | 代码规划：定义 experiments/roundN/ 结构 + run_summary.json |
| python-model-code-generator | Python 代码生成 |
| matlab-model-code-generator | MATLAB / 北太天元 代码生成 |
| code-reviewer | 代码路由：检测语言并交给对应 reviewer |
| python-code-reviewer | Python 代码审查 |
| matlab-code-reviewer | MATLAB / 北太天元 代码审查 |
| result-report-generator | 实验报告 + 最终结果分析 |
| robustness-checker | 稳健性/敏感性/baseline 对比 |
| final-method-explainer | 最终方法详解：假设/符号/目标/约束/求解/评价 |
| figure-table-planner | 图表规划：诊断图/对比图/论文图/附录图四类 |
| math-figure-generator | 图表生成：出版级 matplotlib 图表，SVG 优先 |
| solution-package-builder | 论文材料包：整合建模+结果给论文手 |
| paper-section-writer | 论文写作：仅从已有制品写，不编数字 |
| paper-polisher | 论文润色：句法/时态/hedging/过度宣称/公式一致性 |
| reference-manager | 参考文献管理：BibTeX 生成 + 虚构引用检测 |
| quality-assurance-auditor | 质量审查：流程完整性 + 三条规则 + 反造假 |

# Workspace Convention (updated)

```
project/
├── planning/                   # 全局规划
│   ├── parse/                  # 题目解析
│   ├── classification/         # 题型分类
│   ├── symbol_table.md         # 统一符号表
│   ├── model_assumptions.md    # 全局模型假设
│   ├── question_dependency.md  # 小问依赖关系
│   └── progress_dashboard.md   # 进度看板
├── methods/                    # 建模手区
│   └── Qx/                     # 候选方法/迭代记录/最终方法详解/图表规划
├── code/                       # 编程手区
│   ├── model-code-analyzer.md  # 代码规划文档
│   ├── Qx/                     # Python 代码 (code/Qx/*.py)
│   └── matlab/Qx/              # MATLAB 代码 (code/matlab/Qx/*.m)
├── results/                    # 正式结果区
│   └── Qx/
│       ├── experiments/roundN/  # 实验输出 (figures/tables/metrics/logs/run_summary.json)
│       └── reports/             # 实验报告/最终结果分析/论文材料包
├── robustness/                 # 稳健性分析区
│   └── Qx/                     # 稳健性报告 + 图表
├── paper/                      # 论文手区
│   ├── sections/               # 各章节草稿
│   ├── figures/                # 最终论文图表 (Type 3 + Type 4)
│   ├── refs.bib                # 参考文献
│   ├── main.tex                # 主文件
│   └── qa_report.md            # QA 报告
├── docs/                       # 长期资料库
│   ├── references/             # 方法参考资料
│   ├── past_papers/            # 优秀论文案例
│   └── templates/              # 论文/报告模板
├── workspace/data_raw/         # 原始数据（只读，不修改）
├── workspace/data_clean/       # 清洗后的数据
└── scratch/                    # 临时探索区（不保证可复现）
```

# Workspace Artifact Contract

- Raw data must remain untouched under `workspace/data_raw/`.
- Cleaned data goes under `workspace/data_clean/`.
- Generated code goes under `code/Qx/` (Python) or `code/matlab/Qx/` (MATLAB).
- Experiment outputs go under `results/Qx/experiments/roundN/` with subdirs: `figures/`, `tables/`, `metrics/`, `logs/`, `run_summary.json`.
- Analysis reports go under `results/Qx/reports/`.
- Robustness artifacts go under `robustness/Qx/`.
- Final paper figures go under `paper/figures/`.
- Paper section drafts go under `paper/sections/`.
- Final assembled materials go under `paper/`.

# Figure Type Classification

All figures must be classified into one of four types:
- Type 1 诊断图: For modeler only — internal debugging. NOT for paper.
- Type 2 对比图: Method comparison — MAY appear in paper.
- Type 3 论文图: Final results — MUST appear in paper. Publication-quality required.
- Type 4 附录图: Supplementary — referenced from main text, shown in appendix.

# Modeling Rules

- Parse first, classify second, select methods third.
- Generate 2-4 candidate methods per subquestion — do not commit to the first idea.
- Match method choice to problem type, data, and contest limits.
- Every main model must have a baseline.
- Do not choose complex models only because they look advanced.
- Do not invent missing data, results, or references.
- Final method selection happens AFTER experiments, by the modeler reviewing experiment reports.
- Create global symbol table early — same symbol for same concept across all subquestions.

# Coding Rules

- Make surgical edits.
- Preserve existing style.
- Make the smallest necessary change when reading or editing files.
- Keep code readable and easy to audit.
- Code must output to `results/Qx/experiments/roundN/` structure.
- Always generate `run_summary.json` after execution.
- Save all outputs to files (CSV, JSON, PNG) — do not rely on console output only.
- Use fixed random seeds for reproducibility (SEED = 2026).
- Do not overwrite validated artifacts unless explicitly repairing them.
- When modifying generated code, explain what changed and how to verify it.

# Paper Writing Rules

- Write only from available problem analysis, method plans, results, figures, and robustness checks.
- The solution package (`qx_solution_package_for_writer.md`) is the primary source per subquestion.
- Keep claims proportional to evidence.
- Do not write numerical claims before results exist.
- Avoid filler prose and unsupported conclusions.
- Method description must match the final method explanation, not an early candidate pool.
- Results must come from the final result analysis, not raw experiment outputs.
- Mention eliminated methods to demonstrate thoroughness.
- Polish before final QA (use paper-polisher).

# Verification and QA

- Re-check assumptions, outputs, and narrative consistency before handoff.
- Robustness or sensitivity checks are required before final delivery.
- QA checks both paper quality AND workflow completeness.
- Every major claim must have at least one supporting robustness check or a stated limitation.
- Final delivery requires QA approval.
- Mark blocking issues clearly.
- Return control to the workflow instead of improvising around unresolved problems.
- Do not fabricate data, references, results, figures, or experiments.
- Do not hide uncertainty.

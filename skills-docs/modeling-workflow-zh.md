# 建模工作流

[English](./modeling-workflow.md) | [简体中文](./modeling-workflow-zh.md)

本文档说明 `MathModeling-skills` 中 24 个 skills 在数学建模竞赛中如何串联使用。

我们的目的不是把所有题目都套进同一种解法，而是减少比赛中常见的流程错误：题目没读清就建模，方法还没想清就写代码，结果没跑出来就写论文结论，或者论文中的数字无法追溯到任何结果文件。

## 工作流总览

```text
workflow-orchestrator
→ problem-parser
→ problem-classifier
→ related-paper-analyzer
→ method-selector
→ symbol-table-builder
→ model-assumptions-builder
→ data-auditor-cleaner
→ model-code-analyzer
   ├── python-model-code-generator
   └── matlab-model-code-generator
→ code-reviewer
   ├── python-code-reviewer
   └── matlab-code-reviewer
→ result-report-generator
→ robustness-checker
→ final-method-explainer
→ figure-table-planner
→ math-figure-generator
→ solution-package-builder
→ paper-section-writer
→ paper-polisher
→ reference-manager
→ quality-assurance-auditor
→ workflow-orchestrator
```

每个阶段都应该留下一个可检查的中间产物，供下一个阶段继续使用。

## 分阶段说明

### 0. 全局启动阶段（只跑一次）

**workflow-orchestrator** → 检查当前项目状态，决定下一步。
**problem-parser** → 读题并转化为结构化任务描述。
**problem-classifier** → 判断每个子问题的核心题型。
**related-paper-analyzer** → 分析用户提供的论文，提取可迁移方法。
**symbol-table-builder** → 构建统一的全局符号表。
**model-assumptions-builder** → 提取并组织全局模型假设。

### 每小问迭代循环

```
method-selector → data-auditor-cleaner → model-code-analyzer
  → 代码生成器 → code-reviewer → result-report-generator
  → [建模手审阅实验报告 → 如需修改则回到 method-selector]
  → [或继续锁定最终方法]
→ robustness-checker → final-method-explainer
  → figure-table-planner → math-figure-generator
  → solution-package-builder → paper-section-writer
  → paper-polisher → reference-manager
```

### 最终组装

**quality-assurance-auditor** → 最终 QA 检查。
**workflow-orchestrator** → QA通过后组装终稿。

## 每小问交付链

每个 Qx 完整链路：

1. `method-selector` → `methods/Qx/qx_method_candidates.md`
2. `data-auditor-cleaner` → 清洗数据 + 数据报告
3. `model-code-analyzer` → `code/model-code-analyzer.md`
4. `python/matlab-model-code-generator` → `code/Qx/*` 或 `code/matlab/Qx/*`
5. `code-reviewer` → 审查后代码
6. `result-report-generator` → `results/Qx/experiments/roundN/qx_experiment_report_roundN.md`
7. [循环 1-6 直到方法收敛；建模手记录在 `methods/Qx/qx_method_iteration_log.md`]
8. `robustness-checker` → `robustness/Qx/qx_robustness_report.md`
9. `final-method-explainer` → `methods/Qx/qx_final_method_explanation.md`
10. `result-report-generator`（最终模式）→ `results/Qx/reports/qx_final_result_analysis.md`
11. `figure-table-planner` → `methods/Qx/qx_figure_table_plan.md`
12. `math-figure-generator` → 论文图表
13. `solution-package-builder` → `results/Qx/reports/qx_solution_package_for_writer.md`
14. `paper-section-writer` → `paper/sections/qx.tex`
15. `paper-polisher` → 润色后论文分节
16. `reference-manager` → `paper/refs.bib` + 引用审计
17. 建模手审阅 + 编程手审阅 → Finalized

## 工作流限制

以下限制是有意设置的：

- 题目未解析前，不进入方法选择。
- 候选方法池不存在时，不生成代码。
- 没有结果 artifact 前，不写论文结论。
- Qx 没有 `qx_final_method_explanation.md` 时不准写最终论文（规则 1）。
- Qx 没有 `qx_final_result_analysis.md` 时不准交给论文手（规则 2）。
- Qx 没有 `qx_solution_package_for_writer.md` 时论文手不准开始（规则 3）。
- 没有 baseline 和稳健性证据前，不声称模型更优。
- QA 未通过前，不组装最终论文。
- 原始数据必须保持不变。

## 三条关键规则

规则 1：没有最终方法详解，不准写最终论文。
规则 2：没有最终结果分析，不准交给论文手。
规则 3：论文手只看材料包，不从零散结果里猜。

## Workspace 结构

```
project/
├── planning/                   # 全局规划
│   ├── parse/                  # 题目解析
│   ├── classification/         # 题型分类
│   ├── symbol_table.md         # 统一符号表
│   ├── model_assumptions.md    # 全局模型假设
│   ├── question_dependency.md  # 小问依赖
│   └── progress_dashboard.md   # 进度看板
├── methods/Qx/                 # 每小问方法产物
├── code/Qx/                    # Python 代码
├── code/matlab/Qx/             # MATLAB 代码
├── results/Qx/
│   ├── experiments/roundN/     # 实验输出
│   └── reports/                # 分析报告
├── robustness/Qx/              # 稳健性报告
├── paper/                      # 论文材料
└── workspace/                  # 数据和外部工作区
```

## 回退说明

回退不是失败，而是正常流程的一部分。发现问题时，应回到最早能够修复它的阶段。

| 问题 | 回到 |
|------|------|
| 题目理解错误 | `problem-parser` |
| 题型判断错误 | `problem-classifier` |
| 方法数据不匹配 | `method-selector` |
| 符号跨小问冲突 | `symbol-table-builder` |
| 假设冲突 | `model-assumptions-builder` |
| 数据问题 | `data-auditor-cleaner` |
| 代码失败或与方法不一致 | `code-reviewer` |
| 实验结果不好 | `method-selector`（修改方法）|
| 结论不稳定 | `robustness-checker` |
| 图表没有证据 | `figure-table-planner` |
| 论文结论无支撑 | `paper-section-writer` |
| 措辞问题 | `paper-polisher` |
| 引用未验证 | `reference-manager` |
| 提交有阻塞 | `quality-assurance-auditor` |

## 说明

在有充分理由时，可以跳过或合并某些阶段，但这个选择应该被明确记录。如果一个结果、图表或结论无法追溯到任何 artifact，就应该把它视为未完成。

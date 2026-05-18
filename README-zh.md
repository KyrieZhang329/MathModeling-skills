# MathModeling-skills

[English](./README.md) | [简体中文](./README-zh.md)

一套用于数学建模竞赛的 Claude Code / Codex Skills，把比赛流程拆成可以一步步走完的小阶段

## 项目初衷

数模比赛真正出问题的地方，往往不是"不会模型"，而是：

- 题目没读清楚，做的根本不是题目要求的事
- 一上来就上复杂模型，不先跑一个 baseline 看看
- 代码跑得通，但实际做的事和论文里写的方法对不上
- 论文里的数字在脚本输出里找不到对应
- 论文里写"我们的模型更好"，但既没有 baseline 也没有敏感性分析
- 图画得漂亮，但说不清这张图到底支持哪个结论

基于以上问题，MathModeling-skills系统能够把比赛流程拆成一组小 skill，每个skill只负责一件事，并要求 agent 完成当前阶段的任务后再进入下一阶段

## 项目作用

- 把比赛流程拆成 26 个职责单一的 skill，每个 skill 只做一件事
- 用 6 个显式门控 (Gate G1–G6) 替代"按阶段顺序推进"——每个门有 `enter_condition / pass_criteria / fail_fallback`，不达标不放行；G2 (method→code) 和 G4 (results→paper) 是最关键的两个收敛点
- 用 3 个正交的独立审计层（`consistency-auditor` 跨媒介一致性、`completeness-auditor` 落盘审计、`quality-assurance-auditor` 流程完整性）替代单一 QA——任一审计失败都阻断最终组装，避免 agent 自我宣称"完成"
- 用 `frozen_numbers.json` 不可变快照锁定 results→paper 的数字，防止 bug 修复后论文数字偷偷漂移
- 留档中间产物（解析后的题目、方法计划、PoC 脚本、数据报告、结果、图、冻结快照、论文草稿、审计报告），任何一个数字都能回查到出处
- Claude Code（`.claude/skills/`）和 Codex（`.codex/skills/`）共用同一套 skill

## 项目边界

- 不会一键给你写完整篇论文
- 不会在没有证据时编造数据、结果、引用或图表。证据缺失会被当成 blocker 报出来
- 在结果还没跑出来之前，不会写带数字的结论
- 没有 baseline 和稳健性检查的话，不会写"我们的模型更优"这种话
- 不会改你的原始数据。所有清洗都在副本上做
- 不替你做建模判断。具体用什么方法，最终还是你来决定

## Skills 列表（26 个）

| Skill | 作用 |
| --- | --- |
| `workflow-orchestrator` | 总调度：按 Gate G1–G6 决定下一步；session 开头强制 ping 环境（Python/MATLAB/git） |
| `problem-parser` | 读题，把题目拆成目标、对象、约束、数据、输出和子问题 |
| `problem-classifier` | 给每个子问题打题型标签（评价类、预测类、优化类等） |
| `related-paper-analyzer` | 在最终定方法前，收集并分析相关论文，不编造引用 |
| `method-selector` | 每子问 2–4 个候选方案 + **每个候选必须附 ≤30 行 PoC + 可行性数字**（Gate G2）；失败方案 `[REJECTED]` 自动归档 |
| `symbol-table-builder` | 全局符号表构建：统一所有小问的符号约定 |
| `model-assumptions-builder` | 全局模型假设：区分必要 / 简化，评估影响 |
| `data-auditor-cleaner` | 审计数据，列出问题，输出清洗后的数据和一份数据报告（副本操作，不动原始） |
| `model-code-analyzer` | 代码规划：定义 `experiments/roundN/` 结构 + `run_summary.json` |
| `python-model-code-generator` | 当实现目标是 `python` 时生成 Python 建模代码 |
| `matlab-model-code-generator` | 当实现目标是 `matlab` 时生成 MATLAB / 北太天元兼容代码 |
| `code-reviewer` | 路由 skill：识别脚本族并交给对应语言的审查 skill |
| `python-code-reviewer` | 审查 Python 建模代码，**强制落盘 `code/Qx/reviews/qx_python_review.md` ≥ 5 项明确通过项**（Gate G3） |
| `matlab-code-reviewer` | 审查 MATLAB / 北太天元兼容代码，**强制落盘 ≥ 5 项明确通过项**（Gate G3） |
| `result-report-generator` | 多方法实验报告 + 最终结果分析；失败方案标 `[REJECTED]` 并移到 `workspace/archived/` |
| `robustness-checker` | 跑敏感性、误差、和 baseline 的对比 |
| `final-method-explainer` | 帮助建模手将最终选定方法写成完整的数学建模说明 |
| `figure-table-planner` | 规划真正需要的图表，明确每张图要说明什么 |
| `math-figure-generator` | matplotlib 出版级图表；**强制 `render_check_and_log()` 检测 bbox 重叠 / 出界 / 字号 < 6.5pt**，未通过不允许标 Type 3 |
| `solution-package-builder` | 整合建模手和编程手材料；**强制生成 `frozen_numbers.json` 不可变快照**（Gate G4） |
| `paper-section-writer` | 只用已有制品写；**章节字数下限 + 每数值结果 ≥ 3 类讨论维度**（敏感性 / 物理 / 基线 / 跨题 / 不确定性）（Gate G5） |
| `paper-polisher` | 句法 / 时态 / hedging / 过度宣称 / 公式一致性 |
| `reference-manager` | BibTeX 生成 + 虚构引用检测 |
| **`consistency-auditor`** | **【独立审计层 1/3】跨媒介数字 / 文件名 / 符号 / 参数比对**（tex ↔ code ↔ frozen_numbers.json ↔ symbol_table），任一不一致即 fail |
| **`completeness-auditor`** | **【独立审计层 2/3】所有应落盘的 `*_review.md` / `*_audit.md` 必须存在且 ≥ 5 通过项**，verbal "passed" 不算数 |
| `quality-assurance-auditor` | 【独立审计层 3/3】流程完整性 + 三条 critical rules + 反造假；Gate G6 三审核必须全 PASS 才允许 final assembly |

## 标准流程（含 Gate G1–G6）

```text
workflow-orchestrator (session 开头 ping 环境)
→ problem-parser → problem-classifier → related-paper-analyzer
                                                ↓ Gate G1: PROBLEM_PARSED
→ symbol-table-builder + model-assumptions-builder + data-auditor-cleaner
→ method-selector  (每候选 ≤30 行 PoC + 可行性数字；失败方案 [REJECTED] 归档)
                                                ↓ Gate G2: METHOD_VALIDATED (load-bearing)
→ model-code-analyzer
   ├── python-model-code-generator
   └── matlab-model-code-generator
→ code-reviewer (router)
   ├── python-code-reviewer  (强制落盘 ≥5 通过项)
   └── matlab-code-reviewer  (强制落盘 ≥5 通过项)
                                                ↓ Gate G3: CODE_REVIEWED
→ result-report-generator (实验报告 + 最终结果分析; [REJECTED] 归档)
→ robustness-checker → final-method-explainer
→ figure-table-planner → math-figure-generator (强制 render_check)
→ solution-package-builder  (生成 frozen_numbers.json — 不可变快照)
                                                ↓ Gate G4: RESULTS_FROZEN (load-bearing)
→ paper-section-writer  (字数下限 + 每数值 ≥3 类讨论)
                                                ↓ Gate G5: PAPER_SECTION_READY
→ paper-polisher → reference-manager
→ 【独立审计层】consistency-auditor + completeness-auditor + quality-assurance-auditor
                                                ↓ Gate G6: AUDIT_LAYER_PASSED (三审核全 PASS)
→ final assembly
```

顺序和 Gate 是有意义的。orchestrator 会守住这些线：

- 题没解析完不进方法选择（G1）
- 方法没有可跑 PoC 就不写完整代码（G2 — load-bearing；解决"漂亮方案到代码阶段才崩"）
- 代码没落盘 ≥5 项通过项的 review 文件就不算审查通过（G3）
- 结果没冻结到 `frozen_numbers.json` 就不在论文里写数字（G4 — load-bearing；解决"论文 K=30 但代码 K=12"飘移）
- 章节字数不达标 / 数值结果不到 3 类讨论维度，就不算章节完成（G5）
- 三审核（一致性 / 完整性 / QA）任一 fail 都不允许 final assembly（G6）

在方法选择阶段，不应过早收缩成单一路线。`method-selector` 应该先比较每子问 2–4 个候选方案；每个候选要带可跑的 ≤30 行 PoC，否则 Gate G2 阻断。

如果比赛要求 MATLAB / 北太天元，就把 `implementation.target` 设为 `matlab`，并配合 `beita-tianyuan-compatible`、`avoid-heavy-toolboxes` 这类保守兼容说明。

语言分支只影响代码生成和代码审查。后面的 skill 继续只看 `results/Qx/` 下的产物，不关心它们来自 Python 还是 MATLAB。

## 文档索引

- [Modeling Workflow](docs/modeling-workflow.md)：说明整体流程，以及 Python / MATLAB 代码分支如何接入。
- [Problem Taxonomy](docs/problem-taxonomy.md): task types used by `problem-classifier`.
- [Method Selection Tree](docs/method-selection-tree.md): baseline, main model, and improvement choices.
- [Design Principles](docs/design-principles.md): project-level modeling and workflow principles.
- [Figure and Table Guidelines](docs/figure-table-guidelines.md): how to plan visual evidence.
- [Paper Writing Rules](docs/paper-writing-rules.md): rules for drafting paper sections from artifacts.
- [QA Checklist](docs/qa-checklist.md): final audit checklist before submission.

## 使用方式

开始第一轮对话前，推荐先发送对应的 Initial Prompt：

- 中文对话： [Initial Prompt-zh.md](Initial%20Prompt-zh.md)
- 英文对话： [Initial Prompt.md](Initial%20Prompt.md)

### Claude Code

先克隆仓库，在 Claude Code 里打开。涉及的文件：

- `CLAUDE.md`：项目级规则
- `.claude/skills/<skill-name>/SKILL.md`：各阶段的 skill 定义
- `templates/`：解析结果、方法计划等的输出格式参考
- `examples/`：示例工作流（正在更新中）

第一句话可以这样写：

```text
读一下 CLAUDE.md，然后调用 workflow-orchestrator。我们的数模题目在 workspace/problem/，按阶段顺序走，不要跳步
```

### Codex

思路一样，只是换一下入口：

- `AGENTS.md`：项目级规则。
- `.codex/skills/<skill-name>/SKILL.md`：Codex 版的 skill 集。

第一句话：

```text
读一下 AGENTS.md，从 workflow-orchestrator 开始。我们的数模题目在 workspace/problem/，按阶段顺序走，不要跳步
```

### 建议的 workspace 结构

```text
project/
├── planning/                   # 全局规划 (parse / classification / 符号表 / 假设 / dashboard)
├── methods/Qx/                 # 候选方法 + PoC + 迭代记录 + 最终方法详解 + 图表规划
│   └── poc/                    # 每候选 ≤30 行 PoC 脚本 (Gate G2)
├── code/
│   ├── model-code-analyzer.md
│   ├── Qx/                     # Python 代码
│   │   └── reviews/qx_python_review.md   # 代码审查记录 ≥5 项 (Gate G3)
│   └── matlab/Qx/              # MATLAB 代码 (类似目录结构)
├── results/Qx/
│   ├── experiments/roundN/     # figures / tables / metrics / logs / run_summary.json
│   └── reports/                # 实验报告 + 最终结果分析 + 论文材料包 + frozen_numbers.json (Gate G4)
├── robustness/Qx/              # 稳健性报告 + 图
├── paper/
│   ├── sections/               # 章节草稿 (字数下限 + ≥3 类讨论, Gate G5)
│   ├── figures/                # Type 3 论文图 + Type 4 附录图 (render_check 已通过)
│   │   └── render_check.log
│   ├── audits/                 # 独立审计层产物 (Gate G6)
│   │   ├── cross_media_consistency_audit.md
│   │   ├── completeness_audit.md
│   │   ├── reference_audit.md
│   │   └── polish_report.md
│   ├── refs.bib  main.tex  qa_report.md
├── workspace/
│   ├── data_raw/               # 原始数据 (只读, settings.json 强制 deny 写入)
│   ├── data_clean/             # 清洗后的数据
│   └── archived/<Qx>/<method>_REJECTED_roundN/   # 失败方案归档
└── scratch/                    # 临时探索 (不可复现)
```

1. `workspace/data_raw/` 视为只读，settings.json 的 permissions 已强制 deny 写入。
2. `paper/sections/` 里的每个数字必须能在 `results/Qx/reports/frozen_numbers.json` 中查到——consistency-auditor 会逐项比对。
3. `[REJECTED]` 方案自动归档到 `workspace/archived/`，主代码树永远只有 `[CHOSEN]` 和 `[BACKUP]`。
4. `frozen_numbers.json` 不可手工编辑——要改数字必须走 `解冻 → 修改 → 重冻结` 三步（在 `freeze_change_log.md` 留记录后由 solution-package-builder 重新生成）。

### 示例 prompt

新建一个比赛任务：

```text
用 workflow-orchestrator。我们的题目在 workspace/problem/problem.pdf，数据在 workspace/data_raw/，从 problem-parser 开始。
```

中途接着做：

```text
方法计划在 workspace/problem/method_plan.json，清洗后的数据在 workspace/data_clean/，从 model-code-generator 继续。
```

只想对已有结果做一次稳健性检查：

```text
结果在 workspace/results/，用 robustness-checker 对 baseline_results.csv 做对比，不要重跑主模型。
```

## 当前状态

目前还处于早期第一版，功能还不够完善：

- skill 的 prompt 会随着跑过的题目越来越多继续调。
- templates 和 JSON schema 现在比较粗，是有意留白的。
- `examples/` 目前基本是占位，完整的端到端示例是下一步。

如果你拿它跑了一整套流程，发现哪里有问题，欢迎提个issue或者直接将问题发送到我的邮箱：[zjzhang0424@gmail.com](mailto:zjzhang0424@gmail.com)，不尽感谢！

## 后续计划

- 补充评价类、预测类、优化类、综合类问题的完整示例
- 收紧题目解析、方法计划、图表规划、QA 报告的 JSON schema 
- 写入常见模型族方法卡片（AHP/TOPSIS、回归族、时间序列、MIP/启发式、图模型、仿真等）
- 改进 `code-reviewer` 和 `robustness-checker` 之间的衔接，避免重复检查
- 欢迎大家在 `docs/` 里写一些常见踩坑笔记（数据泄漏、baseline 偏移、图和结果对不上之类）
- 兼容其他AI工具的使用

## 致谢与参考

本项目在设计和开发过程中参考了以下优秀项目：

- **[nature-skills](https://github.com/Yuan1z0825/nature-skills)** — 一套面向 *Nature* 期刊标准学术产出的 Claude Code Skills 集合。本项目中的 `math-figure-generator` skill 借鉴了其 `nature-figure` skill 的图表合约（figure contract）、颜色体系、多面板信息架构和 SVG 优先导出等设计理念。nature-skills 由 [Yuan1z0825](https://github.com/Yuan1z0825) 及社区贡献者维护，基于 MIT 协议开源。
- **[figures4papers](https://github.com/ChenLiu-1996/figures4papers)** — nature-skills 中 `nature-figure` 的生产级绘图脚本来源，发表于 *Nature Machine Intelligence* 及顶级 ML/生物信息学会议。

感谢以上项目的作者和贡献者。

## License

MIT.

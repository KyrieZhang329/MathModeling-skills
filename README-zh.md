# MathModeling-skills

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

- 把整个比赛流程拆成 11 个 skill，每个skill对应一个阶段
- 强制按固定流程：先读题，再分类，再选方法，再写代码，再检查，再写论文
- 留档中间产物（解析后的题目、方法计划、数据报告、结果、图、论文草稿），任何一个数字都能回查到出处
- Claude Code（`.claude/skills/`）和 Codex（`.codex/skills/`）共用同一套 skill

## 项目边界

- 不会一键给你写完整篇论文
- 不会在没有证据时编造数据、结果、引用或图表。证据缺失会被当成 blocker 报出来
- 在结果还没跑出来之前，不会写带数字的结论
- 没有 baseline 和稳健性检查的话，不会写"我们的模型更优"这种话
- 不会改你的原始数据。所有清洗都在副本上做
- 不替你做建模判断。具体用什么方法，最终还是你来决定

## Skills 列表

| Skill | 作用 |
| --- | --- |
| `workflow-orchestrator` | 跟踪当前进度，决定下一步用哪个 skill |
| `problem-parser` | 读题，把题目拆成目标、对象、约束、数据、输出和子问题 |
| `problem-classifier` | 给每个子问题打题型标签（评价类、预测类、优化类等） |
| `method-selector` | 为每个子问题选 baseline、主模型、可选的改进模型 |
| `data-auditor-cleaner` | 审计数据，列出问题，输出清洗后的数据和一份数据报告 |
| `model-code-generator` | 按照已经定好的方法计划生成 Python / MATLAB 代码 |
| `code-reviewer` | 审查并修代码：bug、和方法计划对不上的地方、不必要的复杂度 |
| `robustness-checker` | 跑敏感性、误差、和 baseline 的对比 |
| `figure-table-planner` | 规划真正需要的图表，明确每张图要说明什么 |
| `paper-section-writer` | 只用已经存在的结果起草论文分节 |
| `quality-assurance-auditor` | 提交前最后一遍检查：前后矛盾、缺证据、引用不上的内容 |

## 标准流程

```text
workflow-orchestrator
→ problem-parser
→ problem-classifier
→ method-selector
→ data-auditor-cleaner
→ model-code-generator
→ code-reviewer
→ robustness-checker
→ figure-table-planner
→ paper-section-writer
→ quality-assurance-auditor
→ workflow-orchestrator
```

顺序是有意义的。orchestrator 会尽量守住几条线：

- 题没解析完不进方法选择
- 方法计划没过审不写代码
- 结果文件不存在，就不在论文里写对应数字
- 没有 baseline 和敏感性分析，就不写"我们的模型更好"
- QA 没过不组装最终论文

## 文档索引

- [Modeling Workflow](docs/modeling-workflow.md): how the 11 skills are connected.
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

实际跑比赛时，建议把所有产物放在和 skills 仓库分开的 workspace 里：

```text
workspace/
├── problem/          # 原始题目、解析后的题目 JSON
├── data_raw/         # 原始数据，不动
├── data_clean/       # 清洗后的数据和数据审计报告
├── scripts/          # 生成的 Python / MATLAB 代码
├── results/          # 表格、指标、模型输出
├── figures/          # 最终图表
├── paper_sections/   # 论文分节草稿
└── final_paper/      # 拼装好的论文
```

1. `data_raw/` 视为只读。要改数据，先复制一份到 `data_clean/` 再改
2. `paper_sections/` 里的每一段都应该能追溯到 `results/`、`figures/` 或 `data_clean/` 下的某个文件

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

## License

MIT.

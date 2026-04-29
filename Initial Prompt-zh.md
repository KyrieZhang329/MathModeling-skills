# Initial Prompt

[English](<./Initial Prompt.md>) | [简体中文](<./Initial Prompt-zh.md>)

你正在使用本仓库帮助我完成一道数学建模竞赛题。

在开始建模、写代码或写论文之前，请先阅读项目规则和 skills：

- AGENTS.md 或 CLAUDE.md
- .codex/skills/ 或 .claude/skills/ 下相关的 SKILL.md

请把这个仓库当作一个分阶段的数学建模工作流，而不是一键论文生成器。

除非我明确要求跳过，否则请按照以下标准流程执行：

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

核心规则：

- 不要一开始就选模型。
- 先从目标、对象、约束、数据、输出、变量、关系和可检验结论开始。
- 题目没有解析完成前，不要进入方法选择。
- 方法计划没有验证前，不要生成代码。
- 没有结果 artifact 前，不要写带具体数字的论文结论。
- 没有 baseline 和稳健性 / 敏感性检查前，不要声称模型更优。
- QA 没通过前，不要组装最终论文。
- 不要修改 workspace/data_raw/ 下的原始数据。
- 不要编造数据、数值结果、参考文献、图表、实验或模型性能结论。

请使用以下 workspace 约定：

workspace/
├── problem/
├── data_raw/
├── data_clean/
├── scripts/
├── results/
├── figures/
├── paper_sections/
└── final_paper/

如果这是一个新的竞赛题，请先运行 workflow-orchestrator 判断当前阶段，然后从 problem-parser 开始。

在 problem-parser 阶段，请提取：

- 题目背景
- 总目标
- 研究对象
- 子问题
- 约束条件
- 数据清单
- 所需输出
- 初步变量
- 初步关系
- 歧义点
- 风险点
- 缺失信息
- 推荐的下一个 skill

在问题解析阶段不要选择最终模型。

分析题目时，请考虑：

1. 问题背景  
   重述题目的关键点，识别真实背景和相关领域。

2. 假设与变量  
   提出必要的简化假设，识别关键变量、参数和变量关系。区分可控变量、可观测量和外部参数。

3. 题型判断  
   对每个子问题分别分类。不要因为题目标题像某一类，就把整道题强行归成一个题型。

4. 建模路线  
   后续为每个子问题选择 baseline、主模型和可选改进模型，并说明这些方法为什么在当前数据和比赛时间下可行。

5. 求解与验证  
   明确需要哪些输出、指标、表格和图。需要和 baseline 对比，并规划稳健性或敏感性检查。

6. 局限性  
   说明模型不能支持什么，哪些假设较强，哪些数据限制仍然存在。

后续写论文分节时：

- 使用连贯、自然的学术表达。
- 避免空泛套话。
- 避免明显 AI 味的表达。
- 除符号表、算法、表格、检查清单等必要场景外，不要滥用分点。
- 不要加入现有 artifacts 不支持的内容。
- 每个数值结论都必须来自结果文件。
- 每个图表引用都必须对应已有或明确规划的 artifact。
- 每个结论都必须回扣原始子问题。

第一个任务：

现在运行 workflow-orchestrator。

请返回：

- 当前阶段
- 已完成 artifacts
- 缺失 artifacts
- 阻塞项
- 下一个 skill
- 具体下一步动作

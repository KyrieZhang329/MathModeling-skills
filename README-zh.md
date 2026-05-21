<p align="center">
  <img src="docs/assets/logo.svg" alt="MathModeling-skills" width="640"/>
</p>

<p align="center">
  <a href="./README.md">English</a> ·
  <a href="./README-zh.md"><b>简体中文</b></a> ·
  <a href="./CLAUDE.md">项目规则</a> ·
  <a href="./Initial%20Prompt-zh.md">Initial Prompt</a> ·
  <a href="#%E6%A0%87%E5%87%86%E6%B5%81%E7%A8%8B%E5%90%AB-gate-g1g6">流程图</a> ·
  <a href="#%E6%8C%89%E4%BA%A7%E7%BA%BF%E5%88%86%E7%BB%84%E7%9A%84-26-skill">Skill 索引</a>
</p>

<p align="center">
  <img alt="License" src="https://img.shields.io/badge/license-MIT-2E9E44">
  <img alt="Skills" src="https://img.shields.io/badge/skills-26-1A6FC4">
  <img alt="Gates" src="https://img.shields.io/badge/gates-G1--G6-1A6FC4">
  <img alt="Auditors" src="https://img.shields.io/badge/%E7%8B%AC%E7%AB%8B%E5%AE%A1%E8%AE%A1-3-2E9E44">
  <img alt="Claude Code" src="https://img.shields.io/badge/Claude%20Code-supported-E28E2C">
  <img alt="Codex" src="https://img.shields.io/badge/Codex-supported-E28E2C">
  <img alt="Contest" src="https://img.shields.io/badge/contest-MCM%20%7C%20CUMCM-7B5FD6">
</p>

---

> **一套面向数模竞赛的 skill 工具链。** `MathModeling-skills` 把从原题到可交付论文的全过程拆成 **26 个职责单一的 skill**，置于 **6 个显式门控（Gate G1–G6）** 与 **3 个正交的独立审计层** 之后。论文里的每一个数字都可追溯到不可变快照；每个 reviewer / auditor 必须在磁盘上留下实质性的审计文件；没有任何 skill 可以自我宣称"完成"。

## 为什么造这个

数模竞赛真正出问题的地方，往往不是"不会模型"，而是**流程漂移**：

- 题没读清楚，做的不是题目要求的事
- 没跑 baseline 就上复杂模型
- 代码跑得通，但实际做的事跟论文里写的方法对不上
- 论文里写的数字在脚本输出里找不到对应
- "我们的模型更好"——既没 baseline 也没敏感性分析
- 图画得漂亮，但说不清这张图到底支持哪个结论
- 一次 bug 修复后论文数字偷偷漂移，没人察觉

`MathModeling-skills` 围绕这些失败模式设计。每个 skill 只做一件事；每个 gate 强制收敛后才放行；3 个独立审计器从不同角度检查产出。建模判断仍归用户，但审计链路完全由系统兜底。

## 跟普通工作流的差异

| | 普通"按阶段顺序"工作流 | `MathModeling-skills` |
|---|---|---|
| **推进规则** | "当前阶段完了，下一步" | 每个 gate 有显式 `enter_condition / pass_criteria / fail_fallback`；失败会把所有下游产物标 DIRTY |
| **方法 → 代码边界** | 看数学优雅度 | 每个候选必须附 **≤30 行可跑 PoC + 可行性数字**（Gate G2） |
| **代码审查** | 口头"通过" | 必须落盘 `code/Qx/reviews/qx_<lang>_review.md` 含 **≥ 5 项明确通过项**（Gate G3） |
| **结果 → 论文边界** | 论文每次从最新结果重新读 | 数字锁进不可变 **`frozen_numbers.json`** 含 provenance；改数字必须走 `解冻 → 修改 → 重冻结` 三步（Gate G4） |
| **章节质量** | "看起来 OK" | 章节字数下限 + **每数值结果 ≥ 3 类讨论维度**（Gate G5） |
| **图表质量** | 作者目测 | `render_check_and_log()` 强制 bbox / 出界 / 字号检测，未通过不允许标 Type 3 |
| **最终批准** | 一次 QA | **3 个正交审计器**（一致性 / 完整性 / QA）必须全 PASS——任一 fail 都阻断（Gate G6） |
| **失败方案** | 留在主代码树 | `[REJECTED]` 自动归档到 `workspace/archived/<Qx>/<method>_REJECTED_roundN/` |
| **用户硬规则** | 每次对话都要重复 | 持久化进 `.claude/settings.json`（frozen / data_raw 禁写；git push 询问） |

## 项目边界

- 不会一键给你写完整篇论文。
- 不会在没有证据时编造数据、结果、引用或图表。证据缺失会被当成 blocker 报出来。
- 在结果还没跑出来之前，不会写带数字的结论。
- 没有 baseline 和稳健性检查，不会写"我们的模型更优"。
- 不会改你的原始数据——`data_raw/` 只读。
- 不替你做建模判断。具体用什么方法，最终还是你来决定。

## 标准流程（含 Gate G1–G6）

```text
workflow-orchestrator（session 开头 ping 环境）
 │
 ▼  problem-parser → problem-classifier → related-paper-analyzer
                                          [ Gate G1: PROBLEM_PARSED ]
 ▼  symbol-table-builder + model-assumptions-builder + data-auditor-cleaner
 ▼  method-selector   ── 每候选：≤ 30 行 PoC + 可行性数字
                                          [ Gate G2: METHOD_VALIDATED  ★ load-bearing ]
 ▼  model-code-analyzer
        ├── python-model-code-generator
        └── matlab-model-code-generator
 ▼  code-reviewer（router）
        ├── python-code-reviewer   ── 落盘 qx_python_review.md（≥ 5 通过项）
        └── matlab-code-reviewer   ── 落盘 qx_matlab_review.md（≥ 5 通过项）
                                          [ Gate G3: CODE_REVIEWED ]
 ▼  result-report-generator   ── [REJECTED] 方案自动归档
 ▼  robustness-checker → final-method-explainer
 ▼  figure-table-planner → math-figure-generator   ── 强制 render_check
 ▼  solution-package-builder   ── 生成 frozen_numbers.json（不可变快照）
                                          [ Gate G4: RESULTS_FROZEN   ★ load-bearing ]
 ▼  paper-section-writer   ── 字数下限 + 每数值 ≥ 3 类讨论
                                          [ Gate G5: PAPER_SECTION_READY ]
 ▼  paper-polisher → reference-manager
 ▼  独立审计层（三者必须全 PASS）：
        ├── consistency-auditor    （跨媒介数字 / 文件 / 符号 / 参数）
        ├── completeness-auditor   （所有应落盘的 *_review.md / *_audit.md 必须存在 ≥ 5 通过项）
        └── quality-assurance-auditor  （流程完整性 + 3 条 critical rules + 反造假）
                                          [ Gate G6: AUDIT_LAYER_PASSED ]
 ▼  终稿组装
```

orchestrator 守住这些线：

- **G1** — 题没解析完不进方法选择。
- **G2** — 方法没有可跑 PoC + 可行性数字就不写完整代码。*Load-bearing：解决"漂亮方案到代码阶段才崩"。*
- **G3** — 没有落盘 `qx_<lang>_review.md` 含 ≥ 5 项明确通过项，就不算"代码审查通过"。
- **G4** — 结果没冻结到 `frozen_numbers.json`，就不在论文里写对应数字。*Load-bearing：解决 bug 修复后"论文 K=30 但代码 K=12"的漂移。*
- **G5** — 章节字数不达标 / 数值结果不到 3 类讨论维度，就不算"章节完成"。
- **G6** — 三个正交审计器**必须全 PASS** 才允许 final assembly。单个审计器 ✅ 永远不够。

## 按产线分组的 26 skill

26 个 skill 分成 5 个产线阶段 + 独立审计层。每个 skill 列出 **What it does（做什么）**、**Gate（守哪个门）**、**Outputs（产物落盘点）**、**Key rule（最关键的规则）**。

---

### 第 1 阶段 — 入口与全局规划

打地基：解析题目、给每个子问题打题型标签、收集文献信号、建全局符号表、列全局模型假设、清洗数据。这一阶段做完后，下游所有 skill 都有一个权威的"Qx 在问什么"和"数据是什么"。

#### `workflow-orchestrator`
- **What it does** — 跟踪每个子问题的状态，决定下一步用哪个 skill。
- **Gate** — 按 G1–G6 的 `enter_condition / pass_criteria / fail_fallback` 评估推进资格；session 开头强制 ping 环境（Python / MATLAB / git）。
- **Outputs** — `planning/progress_dashboard.md`；下一步 skill 推荐报告。
- **Key rule** — 三个独立审计器全 PASS 之前，永远不批准 final assembly。

#### `problem-parser`
- **What it does** — 把题目解析为：目标 / 对象 / 约束 / 数据 / 输出 / 子问题。
- **Outputs** — `planning/parse/`。
- **Key rule** — 不做题型分类、不选方法——那些是下游的事。

#### `problem-classifier`
- **What it does** — 给每个子问题打主类型标签（评价 / 预测 / 优化 / 机理 / 分类 / 图 / 仿真 / 数据分析 / 混合）。
- **Outputs** — `planning/classification/`。
- **Key rule** — 跨两类时给主类型 + 次类型，不强行收一边。

#### `related-paper-analyzer`
- **What it does** — 在最终定方法前，收集并分析相关论文 / 报告 / 可借鉴方法。
- **Outputs** — `workspace/papers/related_paper_analysis.md`。
- **Key rule** — 不编造引用，不假造方法来源。

#### `symbol-table-builder`
- **What it does** — 建全局统一符号表，覆盖所有子问题（决策变量 / 参数 / 集合 / 索引 / 输出 / 单位）。
- **Outputs** — `planning/symbol_table.md`。
- **Key rule** — 同义同符号 / 异义异符号——跨子问题严格一致。

#### `model-assumptions-builder`
- **What it does** — 抽取并组织全局模型假设；区分**必要**假设（模型没它就坏）和**简化**假设（没它模型只是近似）。
- **Outputs** — `planning/model_assumptions.md`。
- **Key rule** — 每条假设都要链到具体建模需求，并写出"如果被违反会怎样"。

#### `data-auditor-cleaner`
- **What it does** — 审计原始数据，列出问题，输出清洗后数据 + 数据报告。副本操作——永远不动 `workspace/data_raw/`。
- **Outputs** — `workspace/data_clean/`、`workspace/data_clean/data_report.md`。
- **Key rule** — 原始数据只读（`.claude/settings.json` 的 permissions 在系统层强制）。

---

### 第 2 阶段 — 方法验证（Gate G2）

**Load-bearing** 边界——建模手的想法与编程手的代码之间。这里是数模竞赛最容易翻车的地方：方法看起来漂亮，到真实数据上跑就崩，离截止还有 3 天。Gate G2 强制每个候选在代码生成前先有一份可跑的可行性检查。

#### `method-selector`
- **What it does** — 每子问 2–4 个候选方案；每个候选必须附 **≤ 30 行可跑 PoC** + 在小数据切片上的具体可行性数字。开头向建模手问一个最关键的颗粒度对齐问题。
- **Gate** — G2（METHOD_VALIDATED）。没有 PoC 的候选只是假设，不算候选。
- **Outputs** — `methods/Qx/qx_method_candidates.md` + `methods/Qx/poc/<candidate>_poc.py|.m`。
- **Key rule** — PoC 失败 → 候选标 `[REJECTED]`，PoC 脚本移到 `workspace/archived/`。主 `methods/Qx/poc/` 只留 `[CHOSEN]` 和 `[BACKUP]`。

---

### 第 3 阶段 — 代码执行与审查（Gate G3）

把验证过的方法翻译成可跑代码，运行实验，对照方法计划做审查。reviewer 必须在磁盘上留下实质性 review 文件——口头"通过"会被 `completeness-auditor` 标为 MISSING。

#### `model-code-analyzer`
- **What it does** — 在代码生成前规划好实验目录（`experiments/roundN/`）、`run_summary.json` schema、每个方法的文件布局。
- **Outputs** — `code/model-code-analyzer.md`。
- **Key rule** — 所有实验输出落到 `results/Qx/experiments/roundN/{figures,tables,metrics,logs}/`。

#### `python-model-code-generator`
- **What it does** — 当 `implementation.target = python` 时生成 Python 代码。
- **Outputs** — `code/Qx/*.py`。
- **Key rule** — 输出落盘到文件（CSV / JSON / PNG），不只 print；统一用 `SEED = 2026`。

#### `matlab-model-code-generator`
- **What it does** — 当 `implementation.target = matlab` 时生成 MATLAB / 北太天元兼容代码。默认避开 Live Script、App Designer、重型 toolbox。
- **Outputs** — `code/matlab/Qx/*.m`。
- **Key rule** — 保守 MATLAB 语法；调用 Optimization / Deep Learning / Parallel 等工具箱时显式标兼容风险。

#### `code-reviewer`
- **What it does** — 路由 skill：识别脚本语言，交给 `python-code-reviewer` 或 `matlab-code-reviewer`。
- **Key rule** — 自己不写 review；两种语言并存时双分支都走。

#### `python-code-reviewer`
- **What it does** — 按方法计划审查 Python 代码；surgical 修复路径 / 保存 / shape / 泄漏问题，绝不改数学。
- **Gate** — G3（CODE_REVIEWED）。
- **Outputs** — `code/Qx/reviews/qx_python_review.md` 含 **≥ 5 项明确通过项**（文件:行号 引用）。
- **Key rule** — 凑不出 5 条具体通过项就返 `status: not_run`——不许用泛泛的话凑数。

#### `matlab-code-reviewer`
- **What it does** — 按方法计划审查 MATLAB / 北太天元代码；标出 toolbox / Live-Script / App-Designer 依赖；surgical 修复。
- **Gate** — G3（CODE_REVIEWED）。
- **Outputs** — `code/matlab/Qx/reviews/qx_matlab_review.md` 含 **≥ 5 项明确通过项**。
- **Key rule** — 与 Python reviewer 同——不凑数、不口头。

---

### 第 4 阶段 — 结果整合、稳健性、图表、冻结（Gate G4）

把原始实验输出加工成可交付的论文材料包。这一阶段最后一步——`solution-package-builder` 生成**不可变**的 `frozen_numbers.json`。从这一刻起，论文里每一个数字都必须从这份快照里取。冻结后代码改动了想换数字，必须走 `解冻 → 修改 → 重冻结` 三步。

#### `result-report-generator`
- **What it does** — 每轮生成多方法对比实验报告；建模手锁定最终方法后生成最终结果分析。每个方法标 `[CHOSEN]` / `[BACKUP]` / `[REJECTED]`。
- **Outputs** — `results/Qx/experiments/roundN/qx_experiment_report_roundN.md`；`results/Qx/reports/qx_final_result_analysis.md`。
- **Key rule** — `[REJECTED]` 方案移到 `workspace/archived/<Qx>/<method>_REJECTED_roundN/`，主代码树永远干净。

#### `robustness-checker`
- **What it does** — 设计并跑稳健性 / 敏感性 / 误差 / baseline 对比；区分"稳定结论"和"脆弱结论"。
- **Outputs** — `robustness/Qx/qx_robustness_report.md` 含 ≥ 5 项明确通过项。
- **Key rule** — 扫描区间（参数 ±%、场景数）由题型预设映射，不靠 agent 直觉。

#### `final-method-explainer`
- **What it does** — 帮建模手把最终选定方法写成完整说明：假设 / 符号 / 目标 / 约束 / 求解 / 评价。
- **Outputs** — `methods/Qx/qx_final_method_explanation.md`。
- **Key rule** — 论文的模型描述必须对齐这份文件——不是早期 `qx_method_candidates.md` 里的候选。

#### `figure-table-planner`
- **What it does** — 规划图表，分四类：Type 1 诊断图（仅建模手）/ Type 2 对比图 / Type 3 论文图 / Type 4 附录图。
- **Outputs** — `methods/Qx/qx_figure_table_plan.md`。
- **Key rule** — Type 1 图永远不进论文。

#### `math-figure-generator`
- **What it does** — 出版级 matplotlib 图：语义色板 / 面板逻辑 / SVG 优先。设计哲学来自 `nature-figure`。
- **Outputs** — `paper/figures/qx_*.svg` + `paper/figures/render_check.log`。
- **Key rule** — Type 3 / Type 4 图必须通过 `render_check_and_log()`（bbox 重叠 / 出界 / 字号 < 6.5pt）。未通过不允许标 Type 3。

#### `solution-package-builder`
- **What it does** — 整合最终方法详解 + 最终结果分析 + 图表规划成一份给论文手的材料包；然后生成不可变数字快照。
- **Gate** — G4（RESULTS_FROZEN）。
- **Outputs** — `results/Qx/reports/qx_solution_package_for_writer.md` + **`results/Qx/reports/frozen_numbers.json`**。
- **Key rule** — `frozen_numbers.json` 不可手工编辑。要改数字必须走 `解冻 → 修改 → 重冻结`（在 `freeze_change_log.md` 留原因，重新调用本 skill）。

---

### 第 5 阶段 — 论文起草与独立审计层（Gate G5 + G6）

论文手只从材料包 + 冻结快照写——绝不从零散原始结果里挖。然后独立审计层从三个正交维度做最终检查。**任一审计器 fail 都阻断论文。**

#### `paper-section-writer`
- **What it does** — 从已验证产物起草论文章节。在 Qx 的三条 critical rules（最终方法详解 + 最终结果分析 + 材料包）满足之前，拒绝写 Qx 的核心章节。
- **Gate** — G5（PAPER_SECTION_READY）。
- **Outputs** — `paper/sections/qx.tex`（或 `.md`）。
- **Key rule** — 每章节满足字数下限；每个数值结果配 **≥ 3 类讨论维度**（敏感性 / 物理含义 / baseline 对比 / 跨子问一致性 / 不确定性）中的至少 3 类。

#### `paper-polisher`
- **What it does** — 润色句法 / 时态 / hedging 校准 / 过度宣称识别 / 文档内公式一致性。
- **Outputs** — `paper/audits/polish_report.md`。
- **Key rule** — hedging 强度匹配证据（`demonstrate` → `suggest` → `may reflect`）；标出绝对化 / 无依据因果 / "首次"主张等。

#### `reference-manager`
- **What it does** — 生成 BibTeX；每条引用对照可检索源做交叉验证。
- **Outputs** — `paper/refs.bib` + `paper/audits/reference_audit.md`。
- **Key rule** — 虚构引用是 blocking；每条必须有可验证的 DOI / URL / 源。

#### `consistency-auditor` *(独立审计层 1/3)*
- **What it does** — 跨媒介比对：`paper/sections/*.tex` 里每个数值 / 文件引用 / 符号 / 参数都必须对得上权威源（优先级：`frozen_numbers.json` → 材料包 → 原始 metrics）。
- **Outputs** — `paper/audits/cross_media_consistency_audit.md` 含 ≥ 5 项明确通过项。
- **Key rule** — 数字容差等于论文精度；任一不一致是 BLOCKING，从不降级为 WARNING。

#### `completeness-auditor` *(独立审计层 2/3)*
- **What it does** — 按 audit 注册表扫描所有应当存在的 `*_review.md` / `*_audit.md`；每个必须 ≥ 5 项明确通过项，且不能 stale（冻结快照必须比所引用源文件新）。
- **Outputs** — `paper/audits/completeness_audit.md` 含 ≥ 5 项明确通过项。
- **Key rule** — 口头"通过"不算数。磁盘上找不到 = MISSING = blocking。

#### `quality-assurance-auditor` *(独立审计层 3/3)*
- **What it does** — 审查问题覆盖、三条 critical rules、模型-结果血缘、图表-claim 可追溯、反造假、结论与证据的比例。
- **Gate** — G6（AUDIT_LAYER_PASSED）——只在三个审计器都 PASS 时通过。
- **Outputs** — `paper/qa_report.md` 含 ≥ 5 项明确通过项。
- **Key rule** — 必须看到 `cross_media_consistency_audit.md` 和 `completeness_audit.md` 也是 PASSED 才批准 assembly。三个审计器正交，单个 ✅ 永远不够。

---

## 建议的 workspace 结构

```text
project/
├── planning/                   # 全局规划 (parse / classification / 符号表 / 假设 / dashboard)
├── methods/Qx/                 # 候选方法 + 迭代记录 + 最终方法详解 + 图表规划
│   └── poc/                    # 每候选 ≤30 行 PoC 脚本 (Gate G2)
├── code/
│   ├── model-code-analyzer.md
│   ├── Qx/                     # Python 代码
│   │   └── reviews/qx_python_review.md   # ≥5 项明确通过项 (Gate G3)
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

**workspace 硬规则**

1. `workspace/data_raw/` 只读，`.claude/settings.json` 已强制 deny 写入。
2. `paper/sections/` 里每个数字必须能在 `results/Qx/reports/frozen_numbers.json` 中查到——`consistency-auditor` 会逐项比对。
3. `[REJECTED]` 方案自动归档到 `workspace/archived/`，主代码树永远只有 `[CHOSEN]` 和 `[BACKUP]`。
4. `frozen_numbers.json` **不可手工编辑**——要改数字必须走 `解冻 → 修改 → 重冻结`（在 `freeze_change_log.md` 留记录后由 `solution-package-builder` 重新生成）。

## 上手

第一轮对话前先发对应 Initial Prompt：

- 英文对话：[Initial Prompt.md](Initial%20Prompt.md)
- 中文对话：[Initial Prompt-zh.md](Initial%20Prompt-zh.md)

### Claude Code

```text
读一下 CLAUDE.md，然后调用 workflow-orchestrator。我们的数模题目在 workspace/problem/，按 gate 顺序走，不要跳步。
```

入口文件：
- `CLAUDE.md` — 项目规则（gate / 审计层 / frozen 约定）
- `.claude/skills/<skill-name>/SKILL.md` — 各 skill 定义
- `.claude/settings.json` — 硬规则 guardrail（`data_raw/` 和 `frozen_numbers.json` 禁写，`git push` 询问）

### Codex

```text
读一下 AGENTS.md，从 workflow-orchestrator 开始。我们的数模题目在 workspace/problem/，按 gate 顺序走，不要跳步。
```

入口文件：
- `AGENTS.md` — 项目规则（CLAUDE.md 的 Codex 镜像）
- `.codex/skills/<skill-name>/SKILL.md` — Codex 版 skill 定义

### 示例后续 prompt

- **接着做**：`Q2 round1 实验报告已出。让 workflow-orchestrator 判断是迭代还是锁方法。`
- **只做稳健性**：`让 robustness-checker 跑 Q1，输入在 results/Q1/reports/，baseline 在 results/Q1/experiments/round2/。不要重跑主模型。`
- **触发审计层**：`所有 Qx 章节起草完毕。依次跑 consistency-auditor、completeness-auditor、quality-assurance-auditor。`

## 文档索引

- [CLAUDE.md](CLAUDE.md) — 项目级规则：gate、独立审计层、frozen-numbers 约定。
- [AGENTS.md](AGENTS.md) — Codex 端的项目规则镜像。
- [Implementation targets](docs/implementation-targets.md) — `python` 与 `matlab` 选择与约束。
- [MATLAB / 北太天元 guidelines](docs/matlab-beita-tianyuan-guidelines.md) — 竞赛友好代码的工具箱回避规则。
- 单 skill 指令：[.claude/skills/](.claude/skills/)（Claude Code）或 [.codex/skills/](.codex/skills/)（Codex）。

## 当前状态

目前还是早期版本。skill prompt 会随着跑过的题目越来越多继续调；templates 与 JSON schema 是有意留白；`examples/` 目前基本是占位，端到端示例是下一步。

如果你拿它跑了一整套流程，发现哪里有问题，欢迎提 issue 或者发邮件到 [zjzhang0424@gmail.com](mailto:zjzhang0424@gmail.com)，多谢！

## 后续计划

- 补充评价类、预测类、优化类、综合类问题的端到端示例。
- 收紧题目解析、方法计划、图表规划、审计报告的 JSON schema。
- 加入常见模型族方法卡片（AHP/TOPSIS、回归族、时间序列、MIP/启发式、图模型、仿真等）。
- 改进 `code-reviewer` 与 `robustness-checker` 之间的衔接，减少重复检查。
- 欢迎大家在 `docs/` 里写一些常见踩坑笔记（数据泄漏、baseline 偏移、图和结果对不上之类）。
- 兼容其他 AI 编码 / 写作工具。

## 致谢与参考

- **[nature-skills](https://github.com/Yuan1z0825/nature-skills)** — 面向 *Nature* 期刊学术标准的 Claude Code skill 集合。本项目的 `math-figure-generator` 借鉴了 `nature-figure` 的图表合约、语义色板、多面板信息架构、SVG 优先导出。由 [Yuan1z0825](https://github.com/Yuan1z0825) 维护，MIT License。
- **[figures4papers](https://github.com/ChenLiu-1996/figures4papers)** — nature-figure 的生产级绘图脚本来源，发表于 *Nature Machine Intelligence* 及顶级 ML / 生物信息学会议。

## License

MIT.

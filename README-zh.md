<p align="center">
  <img src="docs/assets/logo.svg" alt="MathModeling-skills" width="640"/>
</p>

<p align="center">
  <a href="./README.md">English</a> ·
  <a href="./README-zh.md"><b>简体中文</b></a> ·
  <a href="./CLAUDE.md">项目规则</a> ·
  <a href="./Initial%20Prompt-zh.md">Initial Prompt</a> ·
  <a href="mailto:zjzhang0424@gmail.com">📧 联系我</a>
</p>

<p align="center">
  <img alt="License" src="https://img.shields.io/badge/license-MIT-2E9E44">
  <img alt="Skills" src="https://img.shields.io/badge/skills-26-1A6FC4">
  <img alt="Claude Code" src="https://img.shields.io/badge/Claude%20Code-supported-E28E2C">
  <img alt="Codex" src="https://img.shields.io/badge/Codex-supported-E28E2C">
</p>

---

> **一套面向数模竞赛的 skill 工具链。** 用 **26 个职责单一的 skill** + **6 个显式门控（Gate G1–G6）** + **3 个正交的独立审计层** 把原题变成可交付论文。论文里每一个数字都可追溯到不可变快照；每个 reviewer 必须在磁盘上留下实质性 audit 文件；没有任何 skill 可以自我宣称"完成"。
>
> 📧 Bug 反馈 / 建议 / 贡献欢迎邮件 **[zjzhang0424@gmail.com](mailto:zjzhang0424@gmail.com)** — 或者直接提 issue。

## 为什么造这个

数模竞赛真正失败的地方，往往不是"不会模型"，而是**流程漂移**：题没读清楚、没跑 baseline 就上复杂模型、论文里写的数字脚本输出找不到、bug 修复后论文数字偷偷漂移没人察觉。`MathModeling-skills` 把这些漂移模式做成"结构上藏不住"的形态。

## 跟普通工作流的差异

| | 普通"按阶段顺序"工作流 | `MathModeling-skills` |
|---|---|---|
| **推进规则** | "当前阶段完了，下一步" | 每个 gate 有 `enter_condition / pass_criteria / fail_fallback`；失败会把所有下游产物标 DIRTY |
| **方法 → 代码边界** | 看数学优雅度 | 每候选必须附 **≤30 行 PoC + 可行性数字**（Gate G2） |
| **代码审查** | 口头"通过" | 必须落盘 `code/Qx/reviews/qx_<lang>_review.md` 含 **≥ 5 项明确通过项**（Gate G3） |
| **结果 → 论文** | 论文每次从最新结果重读 | 不可变 **`frozen_numbers.json`**；改数字必须走 `解冻 → 修改 → 重冻结`（Gate G4） |
| **最终批准** | 一次 QA | **3 个正交审计器**（一致性 / 完整性 / QA）任一 fail 都阻断（Gate G6） |
| **失败方案** | 留在主代码树 | `[REJECTED]` 自动归档到 `workspace/archived/` |

## 标准流程（Gate G1 – G6）

```text
workflow-orchestrator（session 开头 ping 环境）
 ▼  problem-parser → problem-classifier → related-paper-analyzer       [ G1: PROBLEM_PARSED ]
 ▼  symbol-table-builder + model-assumptions-builder + data-auditor-cleaner
 ▼  method-selector   ── 每候选附 ≤30 行 PoC + 可行性数字              [ G2: METHOD_VALIDATED  ★ ]
 ▼  model-code-analyzer → {python,matlab}-model-code-generator
 ▼  code-reviewer（router）→ {python,matlab}-code-reviewer            [ G3: CODE_REVIEWED ]
 ▼  result-report-generator（[REJECTED] 自动归档）
 ▼  robustness-checker → final-method-explainer
 ▼  figure-table-planner → math-figure-generator（render_check）
 ▼  solution-package-builder ── 生成 frozen_numbers.json              [ G4: RESULTS_FROZEN   ★ ]
 ▼  paper-section-writer（字数下限 + 每数值 ≥ 3 类讨论）                [ G5: PAPER_SECTION_READY ]
 ▼  paper-polisher → reference-manager
 ▼  独立审计层（三者必须全 PASS）：
       consistency-auditor · completeness-auditor · quality-assurance-auditor
                                                                       [ G6: AUDIT_LAYER_PASSED ]
 ▼  终稿组装
```

★ = load-bearing 收敛点。数模翻车最多的就是 G2（漂亮方案到代码阶段崩）和 G4（bug 修完论文数字漂移）。

## 按产线分组的 26 skill

### 第 1 阶段 — 入口与全局规划

打地基：解析题目、子问题题型、文献信号、全局符号表、全局假设、清洗数据。

- **`workflow-orchestrator`** — 跟踪每子问题状态，按 G1–G6 评估推进；session 开头 ping 环境。
- **`problem-parser`** — 解析目标 / 对象 / 约束 / 数据 / 输出 / 子问题 → `planning/parse/`。
- **`problem-classifier`** — 每子问题打主题型标签 → `planning/classification/`。
- **`related-paper-analyzer`** — 收集相关论文；不编造引用。
- **`symbol-table-builder`** — 统一全局符号表 → `planning/symbol_table.md`。
- **`model-assumptions-builder`** — 必要 vs 简化假设 → `planning/model_assumptions.md`。
- **`data-auditor-cleaner`** — 副本清洗 + 数据报告；`data_raw/` 只读。

### 第 2 阶段 — 方法验证（Gate G2 ★）

建模手想法与编程手代码之间的 load-bearing 边界。没有 PoC，数模竞赛就在截止前 3 天翻车。

- **`method-selector`** — 每子问 2–4 个候选；**每候选必须附可跑的 ≤ 30 行 PoC + 真实数据上的具体可行性数字**。PoC 失败 → 候选 `[REJECTED]` 自动归档。*产物：*`methods/Qx/qx_method_candidates.md` + `methods/Qx/poc/*`。

### 第 3 阶段 — 代码执行与审查（Gate G3）

把验证过的方法翻译成代码并对照方法计划审查。reviewer 必须留磁盘文件——口头"通过"不算数。

- **`model-code-analyzer`** — 代码生成前规划 `experiments/roundN/` 结构与 `run_summary.json` schema。
- **`python-model-code-generator`** — 当 target = `python` 时生成 `.py`；统一 `SEED = 2026`。
- **`matlab-model-code-generator`** — 生成 MATLAB / 北太天元兼容 `.m`；避开 Live Script、App Designer。
- **`code-reviewer`** — 按脚本语言路由到对应 reviewer。
- **`python-code-reviewer`** — 必须落盘 `code/Qx/reviews/qx_python_review.md` 含 **≥ 5 项明确通过项**（带 file:line）。
- **`matlab-code-reviewer`** — 同上规则，落到 `code/matlab/Qx/reviews/qx_matlab_review.md`。

### 第 4 阶段 — 结果整合、稳健性、图表、冻结（Gate G4 ★）

把原始实验加工成写论文用的材料包，并发出**不可变**数字快照。冻结后 bug 修复要走 `解冻 → 修改 → 重冻结` 三步。

- **`result-report-generator`** — 多方法对比实验报告 + 最终结果分析；标 `[CHOSEN] / [BACKUP] / [REJECTED]`；rejected 移到 `workspace/archived/`。
- **`robustness-checker`** — 敏感性 / 误差 / baseline 对比 → `robustness/Qx/qx_robustness_report.md`（≥ 5 通过项）。
- **`final-method-explainer`** — 锁定方法的完整说明 → `methods/Qx/qx_final_method_explanation.md`。
- **`figure-table-planner`** — Type 1 诊断 / 2 对比 / 3 论文 / 4 附录。Type 1 永远不进论文。
- **`math-figure-generator`** — 出版级 matplotlib；**强制 `render_check_and_log()`**（bbox 重叠 / 出界 / 字号 < 6.5pt）才能标 Type 3。
- **`solution-package-builder`** — 论文材料包 + **生成 `results/Qx/reports/frozen_numbers.json`**。不可手工编辑。

### 第 5 阶段 — 论文起草与独立审计层（Gate G5 + G6）

论文手只从材料包和冻结快照写。独立审计层从三个正交维度做最终检查。**任一审计器 fail 都阻断论文。**

- **`paper-section-writer`** — 起草章节；强制字数下限 + **每数值结果 ≥ 3 类讨论维度**（敏感性 / 物理 / baseline / 跨题 / 不确定）。
- **`paper-polisher`** — 时态 / hedging 校准 / 过度宣称识别 / 文档内公式一致性。
- **`reference-manager`** — BibTeX + 引用可验证性；虚构引用为 blocking。
- **`consistency-auditor`** — 跨媒介：论文每条声明必须对得上 `frozen_numbers.json` / 磁盘文件 / 符号表。
- **`completeness-auditor`** — 所有应存在的 `*_review.md` / `*_audit.md` 必须存在 ≥ 5 通过项且不 stale。
- **`quality-assurance-auditor`** — 流程完整性 + 三条 critical rules + 反造假。**最终门**：只在另外两个审计器也 PASS 时批准 assembly。

## 下载与使用

仓库设计上**放在你当前比赛项目的根目录里用**——这样 `CLAUDE.md` / `AGENTS.md` / `.claude/settings.json` 自动生效，下游 skill 可以往 `methods/` `code/` `results/` `paper/` 写。如果你想全局安装也行。

### 方案 A — 克隆到比赛项目里（推荐）

```bash
# 进入将要存 methods/ code/ results/ paper/ 的文件夹
git clone https://github.com/KyrieZhang329/MathModeling-skills.git .skills-tmp
# 把 skill 文件挪进项目根
mv .skills-tmp/.claude .claude
mv .skills-tmp/.codex .codex
mv .skills-tmp/CLAUDE.md .
mv .skills-tmp/AGENTS.md .
mv .skills-tmp/docs ./skills-docs
rm -rf .skills-tmp
```

克隆完用 **Claude Code** 或 **Codex** 打开当前文件夹——26 个 skill 自动被发现。第一句话：

```text
读一下 CLAUDE.md，然后调用 workflow-orchestrator。我们的题目在 workspace/problem/，按 gate 顺序走，不要跳步。
```

### 方案 B — 全局安装 skill（Claude Code）

```bash
git clone https://github.com/KyrieZhang329/MathModeling-skills.git
cd MathModeling-skills

# 把 26 个 skill 装到用户级 Claude Code skills
mkdir -p ~/.claude/skills
for d in .claude/skills/*/; do
  cp -R "$d" ~/.claude/skills/
done
```

重启 Claude Code。任何项目里都能用到这些 skill。把 `CLAUDE.md` 和 `.claude/settings.json` 复制到每个新比赛项目里，gate 规则和 guardrail 才能生效。

### 方案 C — 全局安装 skill（Codex）

```bash
git clone https://github.com/KyrieZhang329/MathModeling-skills.git
cd MathModeling-skills

# 把 26 个 skill 装到用户级 Codex skills
mkdir -p ~/.codex/skills
for d in .codex/skills/*/; do
  cp -R "$d" ~/.codex/skills/
done
```

重启 Codex。把 `AGENTS.md` 放到每个新比赛项目里，同样的 gate 规则就生效。

### 更新

```bash
cd MathModeling-skills && git pull
# 然后重跑 方案 B 或 C 的 cp 循环
```

### Initial Prompt

新对话第一句话先发对应版本：

- 英文：[Initial Prompt.md](Initial%20Prompt.md)
- 中文：[Initial Prompt-zh.md](Initial%20Prompt-zh.md)

### 示例后续 prompt

- **接着做：**`Q2 round1 实验报告已出。让 workflow-orchestrator 判断是迭代还是锁方法。`
- **只跑稳健性：**`让 robustness-checker 跑 Q1，输入在 results/Q1/reports/，baseline 在 results/Q1/experiments/round2/。不要重跑主模型。`
- **触发审计层：**`所有 Qx 章节起草完毕。依次跑 consistency-auditor、completeness-auditor、quality-assurance-auditor。`

## Workspace 结构

<details>
<summary>展开查看</summary>

```text
project/
├── planning/                   # 解析 / 分类 / 符号表 / 假设 / dashboard
├── methods/Qx/                 # 候选 + 迭代记录 + 最终方法详解 + 图表规划
│   └── poc/                    # 每候选 ≤30 行 PoC（Gate G2）
├── code/
│   ├── Qx/                     # Python 代码；reviews/qx_python_review.md（Gate G3）
│   └── matlab/Qx/              # MATLAB 代码（同构）
├── results/Qx/
│   ├── experiments/roundN/     # figures / tables / metrics / logs / run_summary.json
│   └── reports/                # 实验 + 最终分析 + 材料包 + frozen_numbers.json（Gate G4）
├── robustness/Qx/
├── paper/
│   ├── sections/               # 字数下限 + ≥3 类讨论（Gate G5）
│   ├── figures/                # Type 3 + Type 4（render_check 已过）
│   ├── audits/                 # cross_media / completeness / reference / polish（Gate G6）
│   ├── refs.bib  main.tex  qa_report.md
├── workspace/
│   ├── data_raw/               # 只读（settings.json deny）
│   ├── data_clean/
│   └── archived/<Qx>/<method>_REJECTED_roundN/
└── scratch/                    # 临时探索，不可复现
```

**硬规则：**`data_raw/` 只读 · 论文每个数字都必须出现在 `frozen_numbers.json` · `[REJECTED]` 自动归档 · `frozen_numbers.json` 不可手工编辑。

</details>

## 边界

不会一键写完整篇论文 · 不会编造缺失的数据 / 结果 / 引用 · 结果未出前不写带数字的结论 · 没 baseline 和稳健性不写"我们的模型更优" · 不动原始数据 · 不替你做建模判断。

## 文档

- [CLAUDE.md](CLAUDE.md) — 项目规则（gate / 审计层 / frozen 约定）。
- [AGENTS.md](AGENTS.md) — Codex 端镜像。
- [docs/implementation-targets.md](docs/implementation-targets.md) — `python` vs `matlab` 选择。
- [docs/matlab-beita-tianyuan-guidelines.md](docs/matlab-beita-tianyuan-guidelines.md) — 竞赛友好的 MATLAB 规则。
- 单 skill：[.claude/skills/](.claude/skills/) · [.codex/skills/](.codex/skills/)。

## 联系与贡献

📧 **[zjzhang0424@gmail.com](mailto:zjzhang0424@gmail.com)** — bug、反馈、贡献，或单纯告诉我你跑了真实比赛后的体验。issue 和 PR 也欢迎。

## 致谢

- **[nature-skills](https://github.com/Yuan1z0825/nature-skills)** — `math-figure-generator` 借鉴了 `nature-figure` 的图表合约、语义色板、多面板架构、SVG 优先导出。由 [Yuan1z0825](https://github.com/Yuan1z0825) 维护，MIT。
- **[figures4papers](https://github.com/ChenLiu-1996/figures4papers)** — nature-figure 的生产级绘图脚本来源。

## License

MIT.

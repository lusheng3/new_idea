# new_idea

独立研究项目：**语义先验引导的联邦稳定因果特征发现**（方向 A）。

按 ResearchPilot-Skills 的格式组织，与主资料库 `E:\因果研究\docs\researcha\` 分开维护。

## 目录结构

```
新idea尝试/
├── docs/                     ← ResearchPilot 标准产物
│   ├── idea_report.md        ← 研究报告（Part 1/2/3）
│   ├── user_requirements.md  ← 用户约束（AI 自动维护）
│   └── papers/               ← 相关论文
├── ideas/                    ← 过程留痕（idea 讨论与选题评估）
│   └── YYYY-MM-DD-xxx.md
└── README.md
```

## 当前状态

- 阶段 A（方向探索）进行中：`docs/idea_report.md` Part 1 已起草，状态 PENDING_REVIEW。
- 三层 RQ 候选：RQ1 核心 / RQ2 机制 / RQ3 边界，待确认。

## 记录索引（ideas/ 过程留痕）

| 日期 | 记录 | 结论 |
|---|---|---|
| 2026-11-24 | [选题B1评估](ideas/2026-11-24-选题B1评估.md) | B1（LLM 先验因果发现）不推荐，核心创新已被覆盖 |
| 2026-11-24 | [方向A核查](ideas/2026-11-24-方向A核查.md) | 方向 A（联邦+LLM 语义先验）新颖性开放，值得立项 |
| 2026-11-24 | [方向A问题定义](ideas/2026-11-24-方向A问题定义.md) | 一页问题定义：四态判定 + 方法边界 + Go/No-Go |
| 2026-11-24 | [方向A可行性验证](ideas/2026-11-24-方向A可行性验证.md) | 语义先验救回弱稳定特征，recall 0.875→1.0（8 种子） |
| 2026-11-24 | [方向A冲突gate验证](ideas/2026-11-24-方向A冲突gate验证.md) | conflict gate 兜住噪声先验，D2 local 0.625→0 |
| 2026-11-25 | [真实LLM先验验证](ideas/2026-11-25-真实LLM先验验证.md) | 真实 LLM 先验 12/12 全对，端到端 recall 0.875→1.0 |

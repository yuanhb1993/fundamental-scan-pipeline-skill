# Fundamental Scan Pipeline

`fundamental-scan-pipeline` 是一个可移植的 Codex Skill，用于：

- 针对一个或多个 A股 / 港股 / 美股标的物获取多源数据
- 基于标准化字段生成简洁、结论先行、直接服务投资决策的中文报告

默认输出不面向以下内容：

- 扫描榜单与覆盖率仪表盘
- 冗长的逐项矩阵展示
- 默认仓位、止损、止盈、精确买点
- 用长篇幅重复解释同一结论

## Core Positioning

这个 skill 的默认报告不是“全量研究档案”，而是“投资决策阅读稿”。

它的核心目标是：

1. 先解决标的是谁、数据来自哪里、哪些信息已经核验
2. 再快速回答这家公司值不值得继续看、风险在哪里、当前价格大概处于什么区间
3. 最后给出应该继续跟踪什么，而不是把大量矩阵和原始表格直接塞给读者

## Default Output Contract

默认输出包含两份内容：

### 1. `report_json`

这是主输出，必须先生成。它只保留对投资决策直接有用的结构化字段：

- `identity`
- `data_boundary`
- `core_view`
- `business_essence`
- `financial_snapshot`
- `risk_counterpoints`
- `valuation_timing`
- `tracking_items`
- `sources`

默认不把 `22步矩阵` 和 `LEI 矩阵` 放进主阅读层，它们只作为可选附录存在。

### 2. `report_markdown_zh`

这是阅读版 Markdown，必须由 `report_json` 渲染生成。

默认只保留以下章节：

1. `报告头`
2. `核心结论`
3. `数据边界`
4. `生意本质`
5. `财务抓手`
6. `风险与反证`
7. `估值与时机`
8. `跟踪项`
9. `来源`

## Optional Appendix

如果任务明确要求完整分析底稿，才允许额外输出：

- `analysis_appendix_json`
- `analysis_appendix_markdown_zh`
- `trade_overlay_json`（仅在显式启用时）

附录层可以承载：

- 22 步状态矩阵
- LEI 校验矩阵
- 更完整的原始数据表
- 交易执行叠加层

默认阅读稿不展示这些内容。

## What This Skill Optimizes For

### 1. Decision-First Reading

- 开头先给结论，不让读者翻很久才知道核心判断
- 保留少量关键数字，而不是整页财务表
- 每个章节只保留能改变投资判断的内容

### 2. Evidence With Boundaries

- 明确哪些字段已验证、哪些只是部分验证
- 明确价格日期和财务日期
- 当数据不够时，直接写清边界，不用空话填充

### 3. Concise Multi-Symbol Output

- 单标的：一份完整但精简的阅读稿
- 多标的：先逐标的完整生成，再组合成 batch 报告
- 批量模式也不允许退化成只有一句话摘要表

## 22-Step And LEI Positioning

`22步框架` 和 `LEI 准则` 仍然保留，但在新版 skill 中，它们的定位发生了变化：

- 默认情况下：它们是后台分析骨架，用来约束推理质量
- 明确要求附录时：才外显为矩阵或详细条目

也就是说：

- 默认报告看起来应该像一份可读的投资备忘录
- 而不是一份给模型自查用的展开式分析日志

## Repository Structure

```text
.
├── SKILL.md
├── README.md
├── agents/
│   └── openai.yaml
├── assets/
│   ├── report_json.template.json
│   ├── report_markdown_zh.template.md
│   ├── report_batch_json.template.json
│   └── report_batch_markdown_zh.template.md
└── references/
    ├── workflow.md
    ├── canonical-schema.md
    ├── scoring-and-reporting.md
    ├── api-contracts.md
    └── output-template.md
```

## Reference Files

### [references/scoring-and-reporting.md](./references/scoring-and-reporting.md)

定义简洁版报告契约、默认章节、可选附录和 Markdown 渲染规则。

### [references/output-template.md](./references/output-template.md)

说明如何复用单标的和多标的的简洁模板。

### [assets/report_json.template.json](./assets/report_json.template.json)

单标的简洁 `report_json` 样例。

### [assets/report_markdown_zh.template.md](./assets/report_markdown_zh.template.md)

单标的简洁 `report_markdown_zh` 样例。

### [assets/report_batch_json.template.json](./assets/report_batch_json.template.json)

多标的批量 `report_batch_json` 样例。

### [assets/report_batch_markdown_zh.template.md](./assets/report_batch_markdown_zh.template.md)

多标的批量 `report_batch_markdown_zh` 样例。

## Typical Usage

```text
Use $fundamental-scan-pipeline to generate a concise, decision-first Chinese report for one or more symbols.
```

```text
用 $fundamental-scan-pipeline 生成简洁、结论先行、只保留关键投资信息的中文报告。
```

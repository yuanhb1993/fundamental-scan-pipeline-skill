# Reporting Contract

## Goal

Build one machine-readable report object and one concise Markdown Chinese report from the same data.

The default report is an investment-decision reading version, not a full analytical dump. It should help the reader quickly answer:

- 这家公司值不值得继续看
- 主要风险是什么
- 现在的价格大概处于什么预期区间
- 还缺哪些关键数据需要补

## Required Artifacts

Default outputs:

- `report_json`
- `report_markdown_zh`

Batch-mode composed outputs when multiple symbols are requested:

- `report_batch_json`
- `report_batch_markdown_zh`

Optional outputs only when explicitly enabled:

- `analysis_appendix_json`
- `analysis_appendix_markdown_zh`
- `trade_overlay_json`

## Internal Scaffolding vs Default Output

Use `22步框架` and `LEI 准则` as internal scaffolding by default.

This means:

- they can shape the reasoning
- they can shape the `core_view`, `financial_snapshot`, and `risk_counterpoints`
- they should not be dumped as full matrices in the default reading version unless the user explicitly asks for an appendix

## Required Report Inputs

The report builder should try to populate at least the following inputs:

- `股票代码 / 名称 / 市场`
- `最新可核验价格参考`
- `最近3年财报口径`
- `行业分类`
- `主要竞争对手`
- `营收 / 利润率 / 现金流 / 负债 / 资本效率` 中最关键的几个字段
- `主要风险点`
- `市场预期 / 估值口径`
- `来源与日期`

When a field is unavailable, preserve the missing state explicitly instead of fabricating a value.

## Report JSON Shape

Use `assets/report_json.template.json` as the canonical sample.

Recommended top-level shape:

- `identity`
- `data_boundary`
- `core_view`
- `business_essence`
- `financial_snapshot`
- `risk_counterpoints`
- `valuation_timing`
- `tracking_items`
- `sources`
- `analysis_appendix` optional
- `trade_overlay` optional

## Markdown Structure

Use `assets/report_markdown_zh.template.md` as the canonical sample.

The default section order should be:

1. `# 报告头`
2. `## 核心结论`
3. `## 数据边界`
4. `## 生意本质`
5. `## 财务抓手`
6. `## 风险与反证`
7. `## 估值与时机`
8. `## 跟踪项`
9. `## 来源`

## Rendering Rules

- Lead with the conclusion. Do not bury the main judgment at the bottom.
- Keep only the numbers that materially change the investment conclusion.
- Prefer a short trend statement plus a few key figures over full raw tables.
- Merge repeated caveats into `数据边界` instead of repeating them in every section.
- Keep fact statements and inference statements distinguishable.
- If the report is batch-mode, each symbol should still get a complete concise report before composition.

## Compression Rules

The default reading version should be compressed aggressively:

- each section should usually fit in 3-6 bullets or a short paragraph
- keep only 1-3 top risks
- keep only 1-2 contrarian or counter-evidence points when they matter
- do not list every metric you have; list the ones that affect the decision
- do not repeat the same conclusion in multiple sections

## Guardrails

Default mode must not output the following unless explicitly requested:

- 22-step matrices
- LEI matrices
- full raw financial tables
- position sizing
- stop-loss / stop-profit
- precise entry points
- valuation claims without a time reference and calculation basis

## Placeholder Policy

Use explicit placeholders such as:

- `数据缺失，暂无法确认`
- `以下判断仅基于已获取字段`
- `实时价格未核验，以下为最近可核验价格参考`

Do not convert missing data into positive or negative conclusions without evidence.

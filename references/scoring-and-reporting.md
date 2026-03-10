# Reporting Contract

## Goal

Build one machine-readable report object and one structured Markdown Chinese report from the same data.

The report is generated for one symbol or for each symbol in a symbol list. The primary artifact is `report_json`. The Markdown report is a rendering artifact derived from that JSON.

## Required Artifacts

Default outputs:

- `report_json`
- `report_markdown_zh`

Optional output only when explicitly enabled:

- `trade_overlay_json`

## Entity Resolution Block

Every report must start with an explicit entity resolution block containing:

- original input
- resolved symbol
- resolved company name
- market
- resolution status: `resolved / ambiguous / unresolved`
- resolution evidence

Do not skip this block when the input is a plain company name.

## Required Report Inputs

The report builder should try to populate at least the following inputs:

- `股票代码`
- `中文名称 / 英文名称`
- `市场`
- `行业分类`
- `最近3年财报`
- `主要竞争对手`
- `最新价格 / 30日涨跌 / 市值 / 成交额`
- `营收 / 营收增速 / EBIT / 经营现金流 / 自由现金流 / 负债`
- `ROE / ROIC / WACC / PE`
- `负面消息摘要`
- `管理层讨论与资本配置观察`

When a field is unavailable, preserve the missing state explicitly instead of fabricating a value.

## 22-Step Matrix Contract

The report must contain a `step_status_json` matrix for the full 22-step framework.

Each step should contain at least:

- `step_id`
- `step_title`
- `verification_status`: `已验证 / 部分验证 / 数据缺失待补`
- `key_fields`
- `evidence_refs`
- `summary`

The default report must not silently omit steps 16, 17, or 18. If trade overlay is disabled, these steps must remain present but can state that execution-layer output is not enabled.

## LEI Matrix Contract

The report must contain a `lei_checks_json` matrix.

Each LEI row should contain at least:

- `rule_id`
- `rule_title`
- `status`: `通过 / 不通过 / 未知`
- `key_fields`
- `evidence_refs`
- `summary`

## Report JSON Shape

Recommended top-level shape:

- `symbol`
- `name`
- `market`
- `snapshot_date`
- `data_freshness`
- `report_freshness`
- `entity_resolution`
- `evidence_summary`
- `required_inputs`
- `step_status_matrix`
- `lei_check_matrix`
- `fact_layer`
- `inference_layer`
- `business_essence`
- `financial_perspective`
- `risk_and_moat`
- `management_insight`
- `valuation_and_timing`
- `contrarian_checks`
- `conclusion`
- `next_data_requests`
- `trade_overlay` optional

## Markdown Structure

`report_markdown_zh` should render the same content in a stable structure.

Recommended section order:

1. `# 报告头`
2. `## 必须补充信息`
3. `## 实体解析`
4. `## 22步状态矩阵`
5. `## LEI 校验矩阵`
6. `## 生意本质`
7. `## 财务透视`
8. `## 风险与竞争力`
9. `## 管理层洞察`
10. `## 综合结论`
11. `## 估值与时机`
12. `## 逆向思考`
13. `## 下一步数据需求`
14. `## 附录：来源与证据摘要`

The Markdown report must be structured. Avoid free-form prose dumps without headings.

## Rendering Rules

- Render Markdown from `report_json`, not from a separate free-form pipeline.
- Use explicit causal phrasing when data exists: `因为...，所以...`.
- When data is missing, say which field is missing, why it matters, and how it weakens the conclusion.
- Keep section order stable across one-symbol and multi-symbol outputs.
- Multi-symbol mode should emit one report per symbol, not merge different companies into a single narrative body.
- Keep fact layer and inference layer visibly distinct.

## Guardrails

Default mode must not output the following as verified results:

- total numeric quality scores without a defined score contract
- position sizing
- stop-loss / stop-profit
- precise entry points
- valuation claims without `valuation_calc_json`

If the caller explicitly enables `trade_overlay`, output that content separately and label it as execution-layer inference.

## Placeholder Policy

Use explicit placeholders such as:

- `数据缺失，暂无法确认`
- `财报未补齐，以下判断仅基于已获取字段`
- `竞争对手列表待补充`
- `交易叠加层未启用，本节仅保留分析占位`

Do not convert missing data into positive or negative conclusions without evidence.

## Persistence And Export

Persist both:

- `report_json`
- `report_markdown_zh`

Exports should allow one row per symbol, with Markdown stored as a text field when CSV or tabular export is needed.

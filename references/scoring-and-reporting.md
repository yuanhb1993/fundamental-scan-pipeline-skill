# Reporting

## Goal

Build one machine-readable report object and one human-readable Chinese report from the same data.

The report is generated for one symbol or for each symbol in a symbol list. Do not rely on ad hoc prose as the primary artifact. The primary artifact is `report_json`.

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

## Report JSON Shape

Recommended top-level shape:

- `symbol`
- `name`
- `market`
- `snapshot_date`
- `data_freshness`
- `evidence_summary`
- `required_inputs`
- `business_essence`
- `financial_perspective`
- `risk_and_moat`
- `management_insight`
- `valuation_and_timing`
- `contrarian_checks`
- `conclusion`
- `next_data_requests`

## Chinese Section Order

Render Chinese report text in a fixed order:

1. `必须补充信息`
2. `生意本质`
3. `财务透视`
4. `风险与竞争力`
5. `管理层洞察`
6. `综合结论`
7. `估值与时机`
8. `逆向思考`
9. `下一步数据需求`

## Rendering Rules

- Render Chinese text from `report_json`, not from a separate free-form pipeline.
- Use explicit causal phrasing when data exists: `因为...，所以...`.
- When data is missing, say which field is missing, why it matters, and how it weakens the conclusion.
- Keep section order stable across one-symbol and multi-symbol outputs.
- Multi-symbol mode should emit one report per symbol, not merge different companies into a single narrative body.

## Placeholder Policy

Use explicit placeholders such as:

- `数据缺失，暂无法确认`
- `财报未补齐，以下判断仅基于已获取字段`
- `竞争对手列表待补充`

Do not convert missing data into positive or negative conclusions without evidence.

## Persistence And Export

Persist both:

- `report_json`
- `report_text_zh`

Exports should allow one row per symbol, with the Chinese report text stored as a text column when CSV or tabular export is needed.

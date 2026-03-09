# Scoring And Reporting

## Hard Filters

For `STOCK` rows, the default hard filters are:

- `revenue_usd_m >= min_revenue_usd_m`
- `avg_turnover_usd_m >= min_turnover_usd_m`
- `ebit_margin_pct > 0`
- `operating_cf_usd_m > 0`
- `total_debt_usd_m / revenue_usd_m <= 1`

For `ETF` rows, the default hard filters are:

- `market_cap_usd_m >= min_etf_aum_usd_m`
- `avg_turnover_usd_m >= min_turnover_usd_m`

Exclude US non-common shares before scoring.

## Component Scores

Recommended score families:

- `business_score`
- `financial_score`
- `risk_score`
- `valuation_score`
- `capital_eff_score`
- `lei_score`
- `total_score`

Representative formulas:

- `growth_score`: center around `15%-20%`
- `financial_score`: average of growth, EBIT margin, CFO quality, FCF quality, and debt safety
- `business_score`: moat + pricing power + market leadership
- `valuation_score`: PE anchored near a target band, then adjusted by trend clock
- `risk_score`: penalize accounting risk, negative-news density, weak ESG
- `capital_eff_score`: reward `ROIC > WACC`

Normalize weight maps before computing `total_score`.

## LEI-Style Rule Pack

Typical LEI hit conditions:

- revenue above threshold
- growth in the target range
- positive EBIT margin
- positive operating cash flow
- positive free cash flow
- debt / revenue <= 1
- liquidity above threshold

Turn hit ratio into `0-10` scale for `lei_score`.

## Decision Tags

Recommended defaults:

- `进入深度分析`: hard pass and `total_score >= 7.5`
- `进入观察池`: hard pass and `total_score >= 6.0`
- `暂不纳入`: otherwise

Trend-to-stage mapping should stay separate from decision tags. Example stage labels:

- `UPTREND_2_OCLOCK`
- `VERTICAL_OVERHEAT`
- `RANGE_3_OCLOCK`
- `DOWNTREND_4_TO_6_OCLOCK`

## Risk Flags

Emit compact, human-readable flags such as:

- `财务造假风险需重点排查`
- `ROIC未显著高于WACC`
- `近30天负面事件密度偏高`
- `债务相对收入压力较大`

## Report Structure

Keep the deep report machine-readable as JSON first, then render text as needed.

Recommended Chinese sections:

- `必须补充信息`
- `生意本质`
- `财务透视`
- `风险与竞争力`
- `管理层洞察`
- `综合结论`
- `估值与时机`
- `逆向思考`
- `下一步数据需求`

Persist both:

- `report_json`
- textified export column produced from the same JSON

## Summary Reasons

Persist short reasons alongside each result row, for example:

- `LEI命中 X/Y 项`
- `总分 Z/10`
- explicit hard-fail reasons when the symbol is rejected

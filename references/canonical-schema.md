# Canonical Schema

## Canonical Universe Row

Use one normalized row shape for every source.

Identity:

- `symbol`
- `name`
- `market`
- `asset_type`
- `industry`
- `competitors`
- `key_asset`
- `critical_daily_task`

Price / liquidity:

- `last_price`
- `price_change_30d`
- `avg_turnover_usd_m`
- `market_cap_usd_m`
- `pe_ttm`
- `trend_clock`

Financial core:

- `revenue_usd_m`
- `revenue_growth_pct`
- `ebit_margin_pct`
- `operating_cf_usd_m`
- `free_cf_usd_m`
- `total_debt_usd_m`

Quality / risk:

- `moat_score`
- `pricing_power_score`
- `esg_score`
- `accounting_risk_score`
- `roic_pct`
- `wacc_pct`
- `news_negative_score`

Metadata:

- `updated_at`
- `currency_note`
- `row_source`
- `row_confidence`
- `field_sources_json`
- `field_confidence_json`
- `evidence_updated_at`

## Symbol Normalization

- CN: normalize 6-digit codes to `.SH` or `.SZ`
- HK: normalize numeric codes to `0001.HK` style
- US: keep ticker as-is unless source-specific remapping is needed
- Normalize `ETF` vs `STOCK` before assigning suffix rules

## Evidence Rules

Tracked evidence fields typically include:

- `name`, `industry`
- financial core fields
- `market_cap_usd_m`, `avg_turnover_usd_m`, `pe_ttm`
- `trend_clock`, `last_price`, `price_change_30d`
- quality / risk score fields

Confidence levels:

- `L3`: `>= 0.85`
- `L2`: `>= 0.60`
- `L1`: `>= 0.30`
- `L0`: lower than `0.30`

Representative source heuristics:

- `AKSHARE_CN`: price/liquidity `0.90`, financial core `0.82`, trend `0.85`
- `AKSHARE_HK`: price/liquidity `0.82`, financial core `0.80`
- `YFINANCE`: price/liquidity `0.78`, financial core `0.65`, trend `0.70`
- `SEC_EDGAR_US`: core financials `0.90`
- `ALPHA_VANTAGE`: core financials `0.86`, valuation/name/industry `0.78`

Do not assign high-confidence stock-style financial evidence to ETF placeholder fields.

## Merge Rules

- If `update_missing_only=true`, only replace missing or unverified fields.
- If a fallback fills a previously empty field, update that field's source/confidence without destroying stronger evidence on untouched fields.
- Distinguish true zero from missing. Debt is the critical case:
  - `debt > 0` -> positive debt
  - `debt == 0` with source evidence -> confirmed zero
  - `debt == 0` with no evidence -> missing / unverified

## Related Persistence Objects

Use separate tables or logical entities for:

- `fundamental_universe`: current canonical rows
- `fundamental_scan_tasks`: one scan run + summary + config snapshot
- `fundamental_scan_results`: one evaluated result row per symbol per task
- `fundamental_db_fetch_jobs`: staged refresh job with retry/resume state

Keep result rows denormalized enough for UI and CSV export, but always link them back to the canonical universe row.

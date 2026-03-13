# API Contracts

## Source Refresh

`POST /api/v1/fundamental-scan/universe/import`

Use for source ingestion into the canonical universe.

Typical payload controls:

- `account_id`
- `source_type`
- `markets`
- `asset_types`
- `symbols`
- `refresh_existing`
- `refresh_limit`
- `refresh_offset`
- `update_missing_only`
- source-specific knobs such as `common_stock_only`, `include_financials`, `disable_proxy`

## Report Generation

Recommended generation endpoint:

`POST /api/v1/fundamental-reports/generate`

Important request fields:

- `symbols`
- `company_names` optional
- `markets` optional
- `as_of_date` optional
- `strict_mode` default `true`
- `allow_inference` default `true`
- `include_markdown` default `true`
- `include_appendix` default `false`
- `include_trade_overlay` default `false`

Expected response shape:

- `report_json`
- `report_markdown_zh`
- `report_batch_json` and `report_batch_markdown_zh` when multiple symbols are requested
- `analysis_appendix_json` only when `include_appendix=true`
- `trade_overlay_json` only when `include_trade_overlay=true`

## Report Detail

Recommended report-facing endpoints:

- `GET /api/v1/fundamental-reports`
- `GET /api/v1/fundamental-reports/{symbol}`
- `GET /api/v1/fundamental-reports/export.csv`

Report detail should include:

- concise report fields for the default reading version
- data boundary fields
- sources and dates
- optional appendix artifacts when explicitly enabled

## Export Rules

Exports should support:

- one symbol or multiple symbols
- selected columns
- file names with market / symbol scope / datetime
- optional inclusion of `report_markdown_zh` as a text column
- optional inclusion of appendix artifacts only when explicitly enabled

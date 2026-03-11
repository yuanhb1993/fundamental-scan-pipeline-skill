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

Expected response shape:

- `job_id` or direct summary
- `success_rows`
- `failed_rows`
- `total_rows`
- optional source-specific diagnostics

## Fetch Job Control

`POST /api/v1/fundamental-db/fetch-now`

Important request fields:

- `account_id`
- `markets`
- `symbols`
- `retry_from_job_id`

Job status should expose:

- top-level `status`, `progress_pct`, `current_step`, `error_message`
- stage-level state keyed by stage code
- stage logs / steps
- `retry_of_job_id`
- `report_count` when report build finishes

`GET /api/v1/fundamental-db/fetch-job`

Use to read the latest job or a specific job by id.

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
- `include_trade_overlay` default `false`

Expected response shape:

- `job_id` or synchronous report summary
- `resolved_symbols`
- `unresolved_inputs`
- `report_count`
- `artifacts`: `report_json`, `report_markdown_zh` when synchronous
- `batch_artifacts`: `report_batch_json`, `report_batch_markdown_zh` when multiple symbols are requested

## Report Detail

Recommended report-facing endpoints:

- `GET /api/v1/fundamental-reports`
- `GET /api/v1/fundamental-reports/{symbol}`
- `GET /api/v1/fundamental-reports/export.csv`

Report detail should include:

- canonical identity fields
- freshness fields
- entity resolution block
- evidence summary
- `step_status_matrix`
- `lei_check_matrix`
- `report_json`
- `report_markdown_zh`
- `report_batch_json` and `report_batch_markdown_zh` when the request is multi-symbol
- optional `trade_overlay_json`

## Export Rules

Exports should support:

- one symbol or multiple symbols
- selected columns
- file names with market / symbol scope / datetime
- optional inclusion of `report_markdown_zh` as a text column
- optional inclusion of `trade_overlay_json` only when explicitly enabled

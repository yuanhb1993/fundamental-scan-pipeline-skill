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

## Report Detail

Recommended report-facing endpoints:

- `GET /api/v1/fundamental-reports`
- `GET /api/v1/fundamental-reports/{symbol}`
- `GET /api/v1/fundamental-reports/export.csv`

Report detail should include:

- canonical identity fields
- freshness fields
- evidence summary
- `report_json`
- optional rendered `report_text_zh`

## Export Rules

Exports should support:

- one symbol or multiple symbols
- selected columns
- file names with market / symbol scope / datetime
- optional inclusion of `report_text_zh` as a text column

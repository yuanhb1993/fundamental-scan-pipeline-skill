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

## Manual Scan

`POST /api/v1/fundamental-scan/run`

Use for a direct scan against the current canonical universe. Persist:

- scan config snapshot
- ruleset version / fingerprint
- source snapshot
- summary counts by market

## Scheduled Scan

- `GET /api/v1/fundamental-scan/schedule`
- `PUT /api/v1/fundamental-scan/schedule`
- `POST /api/v1/fundamental-scan/schedule/run-now`

Keep schedule config separate from fetch-job state.

## Results

- `GET /api/v1/fundamental-scan/results`
- `GET /api/v1/fundamental-scan/results/{id}`
- `POST /api/v1/fundamental-scan/results/{id}/push-to-pool`
- `GET /api/v1/fundamental-scan/results/export.csv`

Result rows should include:

- core scores (`lei_score`, `total_score`, etc.)
- decision / stage tags
- `data_updated_at`
- evidence summary
- risk flags
- report data or a detail endpoint that returns it

CSV export should support:

- current page vs all rows
- selected columns
- task id / market / scope in filename
- report text column generated from `report_json`

## Coverage

`GET /api/v1/fundamental-scan/coverage`

Coverage should be computed from canonical rows and return:

- totals
- price coverage
- turnover coverage
- stock financial coverage
- ETF size coverage
- scan-ready coverage
- hard-pass estimate
- `by_market`
- `by_asset_type`
- `missing_reasons_top`
- debt-field status breakdown

## Database View APIs

- `GET /api/v1/fundamental-db/status`
- `GET /api/v1/fundamental-db/filters`
- `GET /api/v1/fundamental-db/results`
- `POST /api/v1/fundamental-db/fetch-now`
- `GET /api/v1/fundamental-db/fetch-job`

Use the database view as a read-optimized surface over the latest scan task. It should expose the latest scan summary, result filters, staged fetch-job progress, and retry/resume state.

## Fetch Job Payload And Status

`POST /api/v1/fundamental-db/fetch-now`

Important request fields:

- `account_id`
- `markets`
- `retry_from_job_id`

Job status should expose:

- top-level `status`, `progress_pct`, `current_step`, `error_message`
- stage-level state keyed by stage code
- stage logs / steps
- `retry_of_job_id`
- `task_id` from the scoring stage when available

# Workflow

## Scope

- Markets: `CN`, `HK`, `US`
- Asset types: `STOCK`, `ETF`
- Canonical sequence: refresh raw data -> normalize + merge evidence -> compute coverage -> run scan -> expose results

## Stage Model

Default staged refresh order:

1. `CN_REFRESH`
2. `US_REFRESH`
3. `HK_REFRESH`
4. `SCAN_SCORE`

Use a market map so that market-scoped runs can skip unrelated stages:

- `CN_REFRESH` -> `CN`
- `US_REFRESH` -> `US`
- `HK_REFRESH` -> `HK`
- `SCAN_SCORE` -> always enabled

Mark unselected stages as `skipped`, not `failed`.

## Source Strategy

Use source adapters that return canonical rows.

Recommended default order:

- CN: `AKSHARE_CN` -> `YFINANCE` fallback
- US: `AKSHARE_US` -> `YFINANCE` fallback -> optional financial补全 via `SEC_EDGAR_US` or `ALPHA_VANTAGE`
- HK: `AKSHARE_HK` -> `YFINANCE` fallback

Record the final source actually used for each stage. If a fallback succeeds, preserve the primary-source failure message in the job log.

## Failure Rules

- Retry market-refresh stages a small fixed number of times.
- Keep stage logs readable: `第N/M次`, source failure message, retry wait, final outcome.
- Allow partial completion. If CN fails but US/HK succeed, continue and run `SCAN_SCORE`.
- Auto-mark stale `RUNNING` jobs as failed when they have exceeded the no-progress timeout.
- Keep retry/resume state in the fetch-job row so a new job can continue from the first failed enabled stage.

## Freshness Model

Track three different clocks:

- Raw-data freshness: `fundamental_universe.updated_at`
- Evidence freshness: `evidence_updated_at`
- Scan freshness: `fundamental_scan_tasks.created_at` / `ended_at`

Do not treat a fresh scan task as proof that the raw universe was refreshed.

## Delivery Flow

After source refresh:

1. Persist canonical rows
2. Update field-level evidence and confidence
3. Recompute coverage / missing reasons against the same canonical fields
4. Run the evaluator over the filtered universe
5. Persist scan task + scan results
6. Expose detail view, database view, and CSV export against the same result set

## Validation Checklist

- Run one `CN`-only or `US`-only fetch and confirm unrelated stages are `skipped`.
- Force one source failure and verify fallback or partial-failure logic.
- Confirm `SCAN_SCORE` can still complete when at least one market stage fails.
- Confirm stale jobs are released automatically.

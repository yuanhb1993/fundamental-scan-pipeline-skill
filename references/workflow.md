# Workflow

## Scope

- Markets: `CN`, `HK`, `US`
- Asset types: `STOCK`, `ETF`
- Core sequence: refresh raw data -> normalize + merge evidence -> prepare report inputs -> build structured report -> render Chinese report text

## Stage Model

Default staged order:

1. `CN_REFRESH`
2. `US_REFRESH`
3. `HK_REFRESH`
4. `REPORT_BUILD`

Use a market map so market-scoped runs can skip unrelated refresh stages:

- `CN_REFRESH` -> `CN`
- `US_REFRESH` -> `US`
- `HK_REFRESH` -> `HK`
- `REPORT_BUILD` -> always enabled if at least one symbol has usable canonical data

Mark unselected stages as `skipped`, not `failed`.

## Source Strategy

Use source adapters that return canonical rows.

Recommended default order:

- CN: `AKSHARE_CN` -> `YFINANCE` fallback
- US: `AKSHARE_US` -> `YFINANCE` fallback -> optional financial backfill via `SEC_EDGAR_US` or `ALPHA_VANTAGE`
- HK: `AKSHARE_HK` -> `YFINANCE` fallback

Record the final source used for each field when possible, not just the stage-level source. If a fallback succeeds, preserve the primary-source failure message in the job log.

## Failure Rules

- Retry market-refresh stages a small fixed number of times.
- Keep stage logs readable: `第N/M次`, source failure message, retry wait, final outcome.
- Allow partial completion. If one market refresh fails, continue with successful symbols and still allow `REPORT_BUILD`.
- Auto-mark stale `RUNNING` jobs as failed when they exceed the no-progress timeout.
- Keep retry/resume state in the fetch-job row so a new job can continue from the first failed enabled stage.

## Freshness Model

Track three different clocks:

- Raw-data freshness: `fundamental_universe.updated_at`
- Evidence freshness: `evidence_updated_at`
- Report freshness: `report_updated_at`

Do not treat a fresh report as proof that all raw fields were freshly refreshed.

## Delivery Flow

After source refresh:

1. Persist canonical rows
2. Update field-level evidence and confidence
3. Prepare report input fields from the canonical row
4. Build `report_json`
5. Render Chinese report text from the same `report_json`
6. Persist report snapshot and expose detail / export endpoints

## Validation Checklist

- Run one `CN`-only or `US`-only fetch and confirm unrelated stages are `skipped`.
- Force one source failure and verify fallback or partial-failure logic.
- Confirm `REPORT_BUILD` can still complete when at least one market stage fails but some symbols remain usable.
- Confirm stale jobs are released automatically.
- Confirm one-symbol and multi-symbol report generation produce the same section order and semantics.

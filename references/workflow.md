# Workflow

## Scope

- Markets: `CN`, `HK`, `US`
- Asset types: `STOCK`, `ETF`
- Core sequence: entity resolve -> refresh raw data -> normalize + merge evidence -> assemble report_json -> render report_markdown_zh -> validate report contract

## Stage Model

Default staged order:

1. `ENTITY_RESOLVE`
2. `CN_REFRESH`
3. `US_REFRESH`
4. `HK_REFRESH`
5. `REPORT_ASSEMBLE`
6. `MARKDOWN_RENDER`
7. `REPORT_VALIDATE`

Use a market map so market-scoped runs can skip unrelated refresh stages:

- `CN_REFRESH` -> `CN`
- `US_REFRESH` -> `US`
- `HK_REFRESH` -> `HK`
- `ENTITY_RESOLVE` -> always enabled
- `REPORT_ASSEMBLE` -> enabled when at least one symbol has usable canonical data
- `MARKDOWN_RENDER` -> enabled when `report_json` exists
- `REPORT_VALIDATE` -> always enabled when any report artifact exists

Mark unselected stages as `skipped`, not `failed`.

## Entity Resolution Rules

- Inputs may be company names, tickers, exchange codes, or mixed lists.
- Every input must resolve to an explicit listed entity or an explicit unresolved/ambiguous state.
- Persist resolution details in `entity_resolution_json`.
- Do not silently convert a company name into a ticker without preserving the resolution chain.

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
- Allow partial completion. If one market refresh fails, continue with successful symbols and still allow `REPORT_ASSEMBLE`.
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
3. Build `entity_resolution_json`
4. Prepare 22-step and LEI evaluation inputs from canonical fields
5. Build `report_json`
6. Render `report_markdown_zh` from the same `report_json`
7. Validate JSON / Markdown alignment and guardrails
8. Persist report snapshot and expose detail / export endpoints

## Guardrails

- Default report mode must not emit trade execution overlays.
- Fact assertions and inference assertions must remain separable.
- Missing data must remain visible in both JSON and Markdown outputs.
- Markdown output must remain structured and section-stable across one-symbol and multi-symbol runs.

## Validation Checklist

- Run one `CN`-only or `US`-only fetch and confirm unrelated stages are `skipped`.
- Force one source failure and verify fallback or partial-failure logic.
- Confirm `REPORT_ASSEMBLE` can still complete when at least one market stage fails but some symbols remain usable.
- Confirm stale jobs are released automatically.
- Confirm one-symbol and multi-symbol report generation produce the same section order and semantics.
- Confirm `report_json` and `report_markdown_zh` remain semantically aligned.
- Confirm disabled trade-overlay content is absent by default.

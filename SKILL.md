---
name: fundamental-scan-pipeline
description: Build or extend a multi-market securities data ingestion and fundamental evaluation workflow. Use when Codex needs to implement, refactor, review, or document systems that fetch A股/港股/美股 stock or ETF data, normalize it into a canonical universe schema, track per-field source/confidence evidence, compute coverage and missing-reason diagnostics, run standardized hard filters and weighted scores, generate Chinese deep-report text, or expose staged refresh, scan, results, and export APIs/UI.
---

# Fundamental Scan Pipeline

Implement a portable workflow for source refresh, canonical normalization, evidence tracking, scoring, and result delivery. Treat raw-data freshness, scan-task freshness, and UI freshness as separate concerns.

## Workflow

1. Fix the slice before editing: source adapter, canonical schema, scoring logic, fetch-job orchestration, coverage, export, or UI/API integration.
2. Read [references/workflow.md](references/workflow.md) when changing refresh stages, retry/resume behavior, fallback order, or partial-failure rules.
3. Read [references/canonical-schema.md](references/canonical-schema.md) when adding fields, changing persistence, merging fallback data, or adjusting evidence/confidence behavior.
4. Read [references/scoring-and-reporting.md](references/scoring-and-reporting.md) when changing hard filters, weights, LEI-style rules, decision tags, or report sections.
5. Read [references/api-contracts.md](references/api-contracts.md) when changing route payloads, coverage responses, job progress data, CSV export, or database result views.

## Operating Rules

- Keep one canonical universe row shape across CN/HK/US and STOCK/ETF. Source adapters map into that shape; they do not invent market-specific result schemas.
- Preserve both row-level and field-level evidence. Do not overwrite higher-confidence data with weaker fallback data unless the existing field is missing.
- Distinguish `pending`, `running`, `completed`, `failed`, and `skipped` stage states. `skipped` is expected for market-scoped refresh jobs.
- Allow partial completion. If one market refresh fails, continue remaining enabled markets and run the scoring stage when existing data is still usable.
- Keep `fundamental_universe.updated_at` tied to successful raw writes. A new scan task id only proves the scoring stage ran.
- Preserve Chinese user-facing report sections and decision tags if the product is Chinese-first.

## Validation

- Run one narrow-market smoke test before widening scope.
- Verify raw data freshness and scan-task freshness separately.
- Verify coverage output explains missing data with explicit reasons rather than a generic "not ready".
- Verify retry/resume starts from the first failed enabled stage and leaves unselected markets as `skipped`.
- Verify CSV export and detail views stay aligned with canonical field names and textified report sections.

## Example Triggers

- "把 A/H/US 基本面扫描能力做成可复用模块"
- "给 fetch-now 增加按市场分阶段刷新、失败重试和断点续跑"
- "新增字段证据等级和未覆盖原因 TopN"
- "把扫描结果、深度报告、CSV 导出统一到同一套标准字段"

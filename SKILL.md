---
name: fundamental-scan-pipeline
description: Build or extend a multi-market securities data-ingestion and structured Chinese deep-report workflow. Use when Codex needs to fetch data for one or more A股, 港股, or 美股 symbols, normalize the data into a reusable canonical schema, preserve source and evidence metadata, and generate machine-readable plus human-readable Chinese deep reports. Do not use this skill for ranking dashboards, coverage KPI systems, or portfolio scoring products unless they directly support symbol data acquisition or report generation.
---

# Fundamental Scan Pipeline

Implement a portable workflow for multi-source market data acquisition, canonical normalization, evidence preservation, and structured Chinese deep-report generation for one or more target symbols.

## Workflow

1. Fix the slice before editing: source adapter, canonical schema, merge/evidence behavior, report schema, report rendering, or ingestion orchestration.
2. Read [references/workflow.md](references/workflow.md) when changing staged refresh, fallback order, retry/resume behavior, or partial-failure rules.
3. Read [references/canonical-schema.md](references/canonical-schema.md) when adding fields, changing symbol normalization, or adjusting source/evidence merge behavior.
4. Read [references/scoring-and-reporting.md](references/scoring-and-reporting.md) when changing structured report sections, report JSON shape, or text rendering rules.
5. Read [references/api-contracts.md](references/api-contracts.md) when changing ingestion or report detail APIs.

## Operating Rules

- Accept one symbol or a list of symbols as the main input.
- Keep one canonical row shape across CN/HK/US and STOCK/ETF. Source adapters map into that shape.
- Preserve row-level and field-level source evidence. Do not overwrite stronger evidence with weaker fallback data unless the field is missing.
- Treat raw-data freshness separately from report freshness.
- Allow partial completion in acquisition jobs. If one market source fails, keep successful symbols and continue report-ready persistence where possible.
- Keep the report machine-readable first, then render a Chinese text form from the same report object.

## Validation

- Run one narrow-market smoke test before widening scope.
- Verify raw writes and evidence timestamps changed after successful ingestion.
- Verify fallback sources do not silently erase higher-confidence fields.
- Verify report JSON and rendered Chinese text stay semantically aligned.
- Verify one-symbol and multi-symbol inputs both work.

## Example Triggers

- "针对一个美股代码生成结构化中文深度报告"
- "针对一组 A/H/US 标的物抓取数据并输出报告"
- "给多市场抓取增加 staged refresh、fallback 和断点续跑"
- "统一不同数据源的 canonical schema 和 evidence 字段"

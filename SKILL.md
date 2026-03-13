---
name: fundamental-scan-pipeline
description: Build or extend a multi-market securities data-ingestion and decision-first Chinese report workflow. Use when Codex needs to resolve one or more A股, 港股, or 美股 targets into explicit listed entities, fetch and normalize their data into a reusable canonical schema, preserve source and evidence metadata, and generate concise, investment-useful `report_json` plus Markdown reports. Do not use this skill for ranking dashboards, coverage KPI systems, portfolio scoring, or trading execution overlays unless the task explicitly asks for them.
---

# Fundamental Scan Pipeline

Implement a portable workflow for multi-source market data acquisition, canonical normalization, evidence preservation, and concise Chinese report generation for one or more target symbols.

## Workflow

1. Fix the slice before editing: entity resolution, source adapter, canonical schema, merge/evidence behavior, report schema, Markdown rendering, or ingestion orchestration.
2. Read [references/workflow.md](references/workflow.md) when changing staged refresh, entity resolution, fallback order, retry/resume behavior, or partial-failure rules.
3. Read [references/canonical-schema.md](references/canonical-schema.md) when adding fields, changing symbol normalization, adjusting source/evidence merge behavior, or extending report persistence fields.
4. Read [references/scoring-and-reporting.md](references/scoring-and-reporting.md) when changing concise report JSON shape, Markdown rendering rules, or optional appendix behavior.
5. Read [references/api-contracts.md](references/api-contracts.md) when changing ingestion, report generation, detail, or export APIs.
6. Read [references/output-template.md](references/output-template.md) when you need reusable sample artifacts or need to keep JSON and Markdown output aligned with the standard template.

## Operating Rules

- Accept one symbol or a list of symbols as the main input.
- Resolve each input into an explicit listed entity before analysis. Persist the mapping in `entity_resolution_json`.
- Keep one canonical row shape across `CN/HK/US` and `STOCK/ETF`. Source adapters map into that shape.
- Preserve row-level and field-level source evidence. Do not overwrite stronger evidence with weaker fallback data unless the field is missing.
- Treat raw-data freshness separately from evidence freshness and report freshness.
- Keep the primary artifact machine-readable first: generate `report_json` before any text rendering.
- Default report mode must be decision-first and compressed. The output should help an investor decide whether to ignore, observe, research deeper, or wait for a better price.
- Treat `22步框架` and `LEI 准则` as internal analytical scaffolding. Do not dump their full matrices into the default reading version unless the user explicitly asks for an appendix.
- Always render a structured Markdown Chinese report from the same `report_json`. Do not generate free-form prose independently.
- Default Markdown should contain only the sections that directly affect investment decisions: `报告头 / 核心结论 / 数据边界 / 生意本质 / 财务抓手 / 风险与反证 / 估值与时机 / 跟踪项 / 来源`.
- Keep only the most decision-relevant numbers. Prefer 4-8 key metrics over full raw statement dumps.
- Separate fact layer from inference layer. Unsupported inferences must not be promoted to verified facts.
- Do not emit position sizing, stop-loss, stop-profit, or entry timing overlays unless the task explicitly enables an optional `trade_overlay` mode.

## Validation

- Run one narrow-market smoke test before widening scope.
- Verify entity resolution is explicit and auditable for each symbol.
- Verify raw writes and evidence timestamps changed after successful ingestion.
- Verify fallback sources do not silently erase higher-confidence fields.
- Verify `report_json` and Markdown output stay semantically aligned.
- Verify one-symbol and multi-symbol inputs both work.
- Verify the default reading version is concise and does not contain 22-step / LEI matrices unless explicitly requested.
- Verify disabled trade-overlay content is absent from the default report.

## Example Triggers

- "针对一个美股代码生成简洁、结论先行的中文投资报告"
- "针对一组 A/H/US 标的物抓取数据并输出逐标的精简深度报告"
- "把默认输出改成只保留对投资决策直接有用的内容"
- "统一不同数据源的 canonical schema、entity resolution 和 evidence 字段"

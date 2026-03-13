# Output Template Guide

## Purpose

Use these templates when you need a concise, decision-first output contract for one-symbol or multi-symbol reports.

Template files:

- `assets/report_json.template.json`
- `assets/report_markdown_zh.template.md`
- `assets/report_batch_json.template.json`
- `assets/report_batch_markdown_zh.template.md`

## Single-Symbol Rule

Single-symbol mode should emit:

- one concise `report_json`
- one concise `report_markdown_zh`

The report should read like an investment memo, not a full analysis log.

## Multi-Symbol Rule

Multi-symbol mode must still generate one complete concise report per symbol first.

Required generation order:

1. Generate one complete concise `report_json` per symbol.
2. Generate one complete concise `report_markdown_zh` per symbol.
3. Compose `report_batch_json` from the full per-symbol JSON artifacts.
4. Compose `report_batch_markdown_zh` by concatenating the full per-symbol Markdown reports in order.

## Minimum Reuse Rules

- Preserve the decision-first structure.
- Keep the Markdown heading order stable.
- Keep only decision-relevant numbers in the default reading version.
- Keep `analysis_appendix` empty unless explicitly enabled.
- Keep `trade_overlay` empty unless explicitly enabled.

## Recommended Adaptation Pattern

- Replace `示例值` with actual resolved values.
- Replace placeholders with actual validated data when available.
- If data is incomplete, say so once in `数据边界`, then avoid repeating the same warning in every paragraph.
- If the user asks for full detail, attach it as appendix instead of bloating the default reading version.

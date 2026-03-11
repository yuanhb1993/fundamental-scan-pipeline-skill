# Output Template Guide

## Purpose

Use these templates when you need a stable, reusable output contract for one-symbol or multi-symbol deep reports.

Template files:

- `assets/report_json.template.json`
- `assets/report_markdown_zh.template.md`
- `assets/report_batch_json.template.json`
- `assets/report_batch_markdown_zh.template.md`

## Single-Symbol Rule

Single-symbol mode should emit:

- one complete `report_json`
- one complete `report_markdown_zh`

## Multi-Symbol Rule

Multi-symbol mode must not collapse output into a summary-only table.

Required generation order:

1. Generate one complete `report_json` per symbol.
2. Generate one complete `report_markdown_zh` per symbol.
3. Compose `report_batch_json` from the full per-symbol JSON artifacts.
4. Compose `report_batch_markdown_zh` by concatenating the full per-symbol Markdown reports in order.

This means batch output is a composition of complete reports, not a shortened list.

## How To Use

1. Build per-symbol `report_json` first from canonical fields, evidence, 22-step state, and LEI checks.
2. Keep placeholders when data is missing. Do not collapse missing fields into silent omissions.
3. Render per-symbol `report_markdown_zh` from the same JSON object.
4. In multi-symbol mode, compose batch artifacts only after all per-symbol artifacts are available or explicitly marked failed.

## Minimum Reuse Rules

- Preserve top-level field names unless you are intentionally versioning the contract.
- Keep the Markdown heading order stable.
- Keep fact layer and inference layer separate.
- Keep `trade_overlay` empty unless explicitly enabled.
- In batch mode, do not replace full reports with compact summaries.

## Recommended Adaptation Pattern

- Replace `示例值` with actual resolved values.
- Replace `evidence_refs` with primary-source references whenever possible.
- Replace placeholder statuses with actual `已验证 / 部分验证 / 数据缺失待补` or `通过 / 不通过 / 未知` values.
- If you add custom sections, add them after the standard sections instead of reordering the standard contract.
- If one symbol fails, keep its failure state explicit in the batch artifact instead of silently removing it.

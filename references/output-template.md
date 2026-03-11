# Output Template Guide

## Purpose

Use these templates when you need a stable, reusable output contract for one-symbol or multi-symbol deep reports.

Template files:

- `assets/report_json.template.json`
- `assets/report_markdown_zh.template.md`

## How To Use

1. Build `report_json` first from canonical fields, evidence, 22-step state, and LEI checks.
2. Keep placeholders when data is missing. Do not collapse missing fields into silent omissions.
3. Render `report_markdown_zh` from the same JSON object.
4. In multi-symbol mode, emit one JSON artifact and one Markdown artifact per symbol.

## Minimum Reuse Rules

- Preserve top-level field names unless you are intentionally versioning the contract.
- Keep the Markdown heading order stable.
- Keep fact layer and inference layer separate.
- Keep `trade_overlay` empty unless explicitly enabled.

## Recommended Adaptation Pattern

- Replace `示例值` with actual resolved values.
- Replace `evidence_refs` with primary-source references whenever possible.
- Replace placeholder statuses with actual `已验证 / 部分验证 / 数据缺失待补` or `通过 / 不通过 / 未知` values.
- If you add custom sections, add them after the standard sections instead of reordering the standard contract.

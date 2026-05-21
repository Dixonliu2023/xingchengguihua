---
name: travel-product-creator
description: Use when designing tourism products, customer-facing itineraries, or sales-ready travel packages from Excel/CSV knowledge files, Feishu Base knowledge, or mixed source materials.
---

# Travel Product Creator

## Overview

This skill designs tourism products from structured knowledge sources and
outputs both customer-facing and internal travel product documents.

Core principle:

1. Do not reason directly on raw Excel or Feishu tables.
2. Normalize knowledge into canonical objects first.
3. Treat Gaode as a validation layer, not a creative layer.
4. Separate customer-facing outputs from internal product outputs.

---

## When to Use

Use this skill when the task is to:

- design a travel product, not just answer travel questions
- turn destination materials into a sellable itinerary
- convert client Excel or CSV files into a tourism product draft
- combine internal Feishu Base knowledge with client project materials
- generate customer-facing itinerary copy plus internal product documents

Do not use this skill for:

- generic travel Q&A with no product design requirement
- live booking confirmation
- final ticketing, inventory locking, or contract issuance
- precise pricing commitments when the pricing layer is not yet connected

---

## Input Modes

The skill supports three source modes:

### 1. `excel_csv`

Use when the client provides:

- `.xlsx`
- `.csv`
- multiple project sheets

Primary references:

- `integrations/canonical-schema.md`
- `integrations/excel-adapter-spec.md`
- `integrations/field-mapping-examples.md`

### 2. `feishu_base`

Use when the knowledge source is a maintained Feishu Base.

Primary references:

- `integrations/canonical-schema.md`
- `integrations/feishu-base-adapter-spec.md`

Implementation note:

- Feishu access should be described as a Base adapter via `lark-cli`
- Do not rely on vague “Feishu MCP” assumptions in output reasoning

### 3. `mixed`

Use when:

- client project data comes from Excel/CSV
- shared reusable knowledge comes from Feishu Base

Priority rule:

- project-specific client material overrides shared public knowledge when they conflict

---

## Canonical Objects

All source data should normalize into these objects before product design:

- `destinations`
- `pois`
- `experiences`
- `resources`
- `risk_warnings`
- `rate_cards` for future pricing support

Reference:

- `integrations/canonical-schema.md`

Do not generate product copy directly from raw table fields if canonical
normalization has not happened.

---

## Execution Order

Always follow this order:

1. detect source mode
2. parse travel demand
3. normalize source data into canonical schema
4. choose quick or deep flow
5. design product strategy
6. validate route feasibility with Gaode
7. generate output documents
8. optionally prepare knowledge write-back candidates

---

## Intake Rules

Build a normalized input context first.

Suggested structure:

```yaml
input_context:
  source_mode: excel_csv | feishu_base | mixed
  route_mode: quick | deep
  destination: "{{目的地}}"
  days: "{{天数}}"
  audience: "{{客群}}"
  budget: "{{预算，可空}}"
  urgency: "{{即时 | 标准 | 宽松}}"
  theme: "{{亲子 | 度假 | 美食 | 文化 | 自驾 | 定制等}}"
```

When key inputs are missing, make reasonable drafting assumptions and mark them
as pending confirmation in the internal logic.

---

## Flow Selection

### Quick Flow

Use when:

- demand is relatively standard
- user needs a first draft quickly
- customer communication is the main goal

Target:

- 3-5 minute quality first draft

Required outputs:

- `templates/simple-itinerary.md`
- `templates/sales-pitch.md`

### Deep Flow

Use when:

- demand is customized or high value
- special audience fit matters
- internal product design depth is required

Required outputs:

- `templates/simple-itinerary.md`
- `templates/sales-pitch.md`
- `templates/product-doc.md`
- `templates/prd-doc.md`

Optional future outputs:

- `quote-sheet.md`
- `cost-breakdown.md`

Reference:

- `skill-refactor-outline.md`

---

## Product Design Layer

This layer defines:

- target audience
- product level
- value proposition
- travel rhythm
- experience priorities
- memory points
- social or emotional hooks

Use these product design principles:

- product = destination + audience + theme + experience
- value = functional value + emotional value + asset value
- not every route is a product; a product needs positioning and memory design
- avoid stacking attractions without narrative or rhythm

Important:

- this layer chooses what kind of product it is
- this layer does not validate distance or transfer feasibility

---

## Gaode Validation Layer

Gaode is a reality-check layer.

Gaode should do:

- resolve POI coordinates
- validate transfer distance and time
- reduce backtracking
- identify infeasible daily route intensity
- support route order optimization

Gaode should not do:

- define theme
- decide emotional value
- write sales copy
- replace product strategy

Expected output shape:

```yaml
validated_route_plan:
  status: valid | needs_adjustment
  daily_routes: []
  rejected_candidates: []
  optimization_notes: []
```

When a route is only partially validated, be explicit about assumptions instead
of pretending certainty.

---

## Customer-Facing Output Contract

The customer-facing main document is:

- `templates/simple-itinerary.md`

It should follow a finished handout style, not a bare schedule table.

It should typically contain:

1. product title and opening copy
2. emotional destination intro
3. product-manager narrative
4. “what should travel be like” section
5. product-designer explanation
6. quick-look itinerary table
7. destination highlights / clock-in section
8. cuisine recommendation
9. accommodation highlights
10. detailed day-by-day itinerary
11. arrival / departure notes
12. service standards
13. exclusions
14. travel notes
15. warm closing copy

Supporting customer-facing document:

- `templates/sales-pitch.md`

This document should:

- help sales explain why the product is worth buying
- summarize ideal audience
- compress route highlights
- provide objection-handling guidance
- stay aligned with the customer-facing itinerary tone

Important:

- customer-facing outputs should read like finished product copy
- do not expose internal raw assumptions, technical schema terms, or adapter logic

---

## Internal Output Contract

Internal outputs are for product, sales, and operations alignment.

### `templates/product-doc.md`

Use for:

- product positioning
- audience fit
- resource decomposition
- experience logic
- route rationale
- risk notes
- future pricing placeholders

### `templates/prd-doc.md`

Use for:

- execution alignment
- internal standards
- structured handoff to team members

Internal outputs should be more explicit than customer outputs about:

- assumptions
- validation status
- pending confirmations
- route risks
- source origin

---

## Knowledge Write-Back Rule

Knowledge write-back is optional and should be conservative.

Only prepare write-back candidates when:

- a highlight is clearly new
- it does not already exist in the knowledge base
- it is marked as unverified until confirmed

Do not treat draft creativity as verified long-term knowledge.

---

## Pricing Reservation

Pricing is not required for every run yet, but the architecture must leave room
for it.

Reserved canonical object:

- `rate_cards`

Reserved future outputs:

- `quote-sheet.md`
- `cost-breakdown.md`

Until pricing is fully connected:

- do not present precise final prices as confirmed
- if price hints exist, present them as reference only

---

## Output Summary Format

At completion, summarize output like this:

```text
✅ 产品创作完成

客户侧输出：
1. [产品名称]-简易行程表.md
2. [产品名称]-销售推介.md

内部输出：
3. [产品名称]-产品文档.md（深度流）
4. [产品名称]-PRD.md（深度流）

数据来源：
- source_mode: excel_csv / feishu_base / mixed

验证状态：
- 路线验证：已验证 / 部分验证 / 待验证
- 待确认项：X项

后续建议：
- {{下一步建议}}
```

---

## References

- `integrations/canonical-schema.md`
- `integrations/excel-adapter-spec.md`
- `integrations/feishu-base-adapter-spec.md`
- `integrations/field-mapping-examples.md`
- `skill-refactor-outline.md`
- `templates/simple-itinerary.md`
- `templates/sales-pitch.md`
- `templates/product-doc.md`
- `templates/prd-doc.md`
- `CHANGELOG.md`

---

*Document version: v2.0*  
*Last updated: 2026-05-21*

# Skill Refactor Outline

## Goal

Refactor `travel-product-creator` from a monolithic workflow note into a stable
tourism product production system that supports:

- Excel / CSV knowledge input
- Feishu Base knowledge input via `lark-cli`
- mixed source mode
- canonical schema normalization
- independent Gaode validation layer
- future pricing extension

---

## Core Principles

1. Business logic should not depend directly on Feishu or Excel
2. All sources must normalize into canonical schema first
3. Gaode is a validation layer, not a creative layer
4. Document generation consumes canonical objects only
5. Quick and deep flows should share the same data foundation

---

## Proposed Directory Shape

```text
travel-product-creator/
├─ SKILL.md
├─ skill-refactor-outline.md
├─ integrations/
│  ├─ canonical-schema.md
│  ├─ excel-adapter-spec.md
│  ├─ field-mapping-examples.md
│  ├─ feishu-base-adapter-spec.md
│  ├─ gaode-validation-spec.md
│  └─ source-selection-rules.md
├─ workflows/
│  ├─ intake/
│  │  ├─ source-detection.md
│  │  └─ demand-normalization.md
│  ├─ quick-flow/
│  │  ├─ planner.md
│  │  └─ doc-generator.md
│  ├─ deep-flow/
│  │  ├─ product-design.md
│  │  ├─ route-design.md
│  │  ├─ commercial-design.md
│  │  └─ risk-design.md
│  └─ validation/
│     └─ gaode-route-validation.md
├─ templates/
│  ├─ simple-itinerary.md
│  ├─ sales-pitch.md
│  ├─ prd-doc.md
│  └─ product-doc.md
└─ references/
   ├─ routing-rules.md
   ├─ category-normalization.md
   ├─ emotion-tag-guide.md
   └─ output-contract.md
```

---

## Architecture Layers

### 1. Intake Layer

Responsibilities:

- detect source mode
- parse user demand
- choose quick vs deep flow

Example object:

```yaml
input_context:
  source_mode: excel_csv | feishu_base | mixed
  route_mode: quick | deep
  destination: 广州
  days: 3
  audience: 亲子
  budget: null
  urgency: standard
```

### 2. Knowledge Adapter Layer

Responsibilities:

- read source data
- normalize into canonical schema

Outputs:

- `destinations`
- `pois`
- `experiences`
- `resources`
- `risk_warnings`
- `rate_cards`

### 3. Product Design Layer

Responsibilities:

- product positioning
- audience fit
- theme and value proposition
- experience selection logic

This layer should not perform route validation.

### 4. Validation Layer

Responsibilities:

- location resolution
- transfer validation
- backtrack checks
- candidate route optimization

This layer should only validate physical feasibility.

### 5. Output Layer

Responsibilities:

- generate stable output documents
- avoid making new strategy decisions

---

## Input Modes

### Mode A: Excel / CSV

Best for:

- client-supplied project knowledge
- one-off product creation
- early-stage drafting

### Mode B: Feishu Base

Best for:

- long-term managed knowledge base
- team-maintained resources
- repeatable internal workflows

### Mode C: Mixed

Best for:

- client-specific material plus internal shared knowledge

Merge rule:

```yaml
priority:
  project_excel: 1
  shared_feishu_base: 2
```

---

## Gaode Layer Positioning

Gaode should be separated as a reality-check layer.

Gaode does:

- resolve coordinates
- validate transfer time and distance
- check route logic
- reduce backtracking

Gaode does not:

- choose product theme
- decide sales angle
- classify emotional value
- generate copy

Suggested output:

```yaml
validated_route_plan:
  status: valid | needs_adjustment
  daily_routes: []
  rejected_candidates: []
  optimization_notes: []
```

---

## Quick Flow

Purpose:

- produce a useful first draft in 3-5 minutes

Fixed steps:

1. intake
2. adapter
3. lightweight product strategy
4. Gaode validation
5. generate documents

Required outputs:

- `simple-itinerary.md`
- `sales-pitch.md`

---

## Deep Flow

Purpose:

- produce a robust internal product package

Fixed steps:

1. intake
2. adapter
3. product positioning
4. experience design
5. route design
6. commercial strategy
7. risk design
8. Gaode validation
9. generate documents

Required outputs:

- `simple-itinerary.md`
- `sales-pitch.md`
- `product-doc.md`
- `prd-doc.md`

Optional outputs:

- `import-report.md`
- `route-validation-report.md`

---

## Output Contract

```yaml
output_contract:
  quick:
    required:
      - simple-itinerary.md
      - sales-pitch.md
    optional: []

  deep:
    required:
      - simple-itinerary.md
      - sales-pitch.md
      - product-doc.md
      - prd-doc.md
    optional:
      - import-report.md
      - route-validation-report.md
      - quote-sheet.md
      - cost-breakdown.md
```

`quote-sheet.md` and `cost-breakdown.md` are reserved for future pricing
support.

---

## Template Fixes

The current skill references `templates/product-doc.md` but the template is not
present. This must be fixed during refactor.

Template roles:

- `simple-itinerary.md`: customer-readable draft
- `sales-pitch.md`: differentiated sales narrative
- `product-doc.md`: internal product strategy record
- `prd-doc.md`: execution-oriented internal specification

---

## Data Standardization References

Add these references to stabilize downstream reasoning:

- `category-normalization.md`
- `emotion-tag-guide.md`
- `output-contract.md`

These files should define:

- standard category vocabulary
- standard emotional-value vocabulary
- stable document guarantees

---

## Pricing Reservation

Pricing is not the immediate implementation target, but the architecture should
reserve a dedicated pricing layer.

Recommended future chain:

```text
adapter -> canonical schema -> product design -> gaode validation -> pricing layer -> docs
```

The canonical object `rate_cards` should be included now even if the first
implementation leaves it empty.

Future pricing outputs:

- `quote-sheet.md`
- `cost-breakdown.md`

---

## Implementation Priority

### P0

- add `canonical-schema.md`
- add `excel-adapter-spec.md`
- add `field-mapping-examples.md`
- add `feishu-base-adapter-spec.md`
- unify output contract
- add missing `product-doc.md`

### P1

- rewrite `SKILL.md` around source adapters and canonical schema
- isolate Gaode validation into its own workflow spec
- add category and emotion standardization

### P2

- support mixed mode formally
- add import reporting output
- prepare pricing-layer specs

# Feishu Base Adapter Spec

## Purpose

This adapter defines how to read knowledge from Feishu Base via `lark-cli` and
normalize it into the canonical schema for `travel-product-creator`.

It replaces vague "Feishu MCP" assumptions with an explicit Base adapter model.

---

## Adapter Role

The Feishu Base adapter is a data access and normalization layer.

It should:

- locate the target Base
- identify the correct tables
- inspect fields
- read records
- normalize values
- map records into canonical objects
- emit an import report

It should not:

- perform product design
- validate routes
- write sales copy
- calculate prices

---

## Dependencies

- `lark-cli`
- [lark-shared](C:/Users/Administrator/.agents/skills/lark-shared/SKILL.md)
- [lark-base](C:/Users/Administrator/.agents/skills/lark-base/SKILL.md)

---

## Supported Modes

### Single Base mode

One Base contains all working tables. This is the recommended mode.

### Split Bases mode

Multiple Bases store different domains like experiences, resources, or risks.

### Mixed mode

Feishu Base acts as a shared knowledge source while client Excel or CSV acts as
project-specific source material.

---

## Table Detection

Recommended table aliases:

```yaml
table_detection:
  destinations:
    aliases: ["destinations", "目的地", "城市主表"]

  pois:
    aliases: ["pois", "景点", "地点", "poi表"]

  experiences:
    aliases: ["experiences", "玩法", "活动", "体验"]

  resources:
    aliases: ["resources", "资源", "供应商", "酒店", "餐厅"]

  risk_warnings:
    aliases: ["risk_warnings", "风险", "风险预警"]

  rate_cards:
    aliases: ["rate_cards", "价格卡", "报价基础", "价格表"]
```

---

## Output Contract

```yaml
feishu_adapter_output:
  canonical_data:
    destinations: []
    pois: []
    experiences: []
    resources: []
    risk_warnings: []
    rate_cards: []

  import_report:
    source_type: feishu_base
    bases_processed: []
    tables_processed: []
    row_counts: {}
    warnings: []
    errors: []
    dropped_rows: []
```

---

## Read Flow

```yaml
read_flow:
  1: locate_base
  2: identify_tables
  3: inspect_fields
  4: read_records
  5: normalize_records
  6: map_to_canonical_schema
  7: emit_import_report
```

### 1. locate_base

- if the user provides a Base link, resolve it directly
- if the user provides a Base name, search for the matching Base
- record `base_token` after resolution

### 2. identify_tables

- list all tables
- map them into canonical roles by alias or explicit configuration

### 3. inspect_fields

- read field structures
- build field mappings
- detect missing required fields

### 4. read_records

- read records using list, search, or query methods

### 5. normalize_records

- trim whitespace
- normalize arrays and enums
- normalize dates

### 6. map_to_canonical_schema

- create standard objects from Feishu rows

### 7. emit_import_report

- summarize rows read, warnings, and dropped data

---

## Business Actions vs Implementation

The top-level skill should talk in business actions, not raw CLI commands.

Business actions:

- `retrieve destination profiles`
- `retrieve poi candidates`
- `retrieve experience candidates`
- `retrieve resources`
- `retrieve risk warnings`
- `write new highlight candidate`

Implementation layer:

- `lark-cli base +record-list`
- `lark-cli base +record-search`
- `lark-cli base +record-get`
- `lark-cli base +data-query`
- `lark-cli base +record-upsert`

---

## Field Mapping Model

Field names in Base may differ from canonical schema, so mapping must remain
configurable.

Example:

```yaml
field_mapping:
  experiences:
    city: city
    title: experience_title
    category: experience_category_raw
    description: experience_description
    duration: duration_hours
    suitable_for: suitable_for
    emotional_value: emotional_value
    highlights: highlights
    risk_warning: embedded_risk_warning
    validation_status: validation_status
```

---

## Cleaning Rules

Use the same cleaning philosophy as the Excel adapter where possible.

```yaml
global_cleaning_rules:
  trim_whitespace: true
  empty_as_null: true

  array_fields:
    split_if_text:
      - "，"
      - "、"
      - ","
      - "/"
      - ";"
      - "；"

  severity_mapping:
    低: low
    中: medium
    高: high

  validation_status_mapping:
    待验证: pending
    已验证: verified
    已拒绝: rejected
```

Feishu-specific rules:

- multi-select fields should be treated as arrays directly
- attachment fields should keep references, not download binary by default
- formula and lookup fields are readable but should not be treated as canonical
  write targets

---

## Minimum Table Requirements

### destinations

Required:

- `destination_id`
- `province`
- `city`

If `destination_id` is missing:

- auto-generate `dest_<city_pinyin>`

### pois

Required:

- `city`
- `poi_name`

If `poi_id` is missing:

- auto-generate `poi_<city_pinyin>_<name_pinyin>`

### experiences

Required:

- `city`
- `experience_title`

If `experience_id` is missing:

- auto-generate `exp_<city_pinyin>_<title_pinyin>`

### resources

Required:

- `city`
- `resource_type`
- `resource_name`

### risk_warnings

Required:

- `city`
- `risk_type`
- `risk_description`

### rate_cards

Required:

- `city`
- `item_type`
- `item_name`
- `pricing_unit`
- `base_price`

If `rate_id` is missing:

- auto-generate `rate_<city_pinyin>_<item_name_pinyin>`

---

## Mixed Mode Merge Rules

When Feishu Base and Excel are both active:

```yaml
merge_priority:
  project_specific_excel: 1
  shared_feishu_base: 2
```

Practical rules:

- project-specific Excel overrides shared Base
- verified records override pending records
- more complete records override sparse ones
- newer verification dates override older ones

---

## Write-back Guidance

If write-back is enabled later, do not write draft highlights directly into the
formal `experiences` table.

Prefer dedicated tables such as:

- `new_highlights`
- `candidate_highlights`

Suggested minimum fields:

- `title`
- `city`
- `description`
- `source`
- `original_product_id`
- `validation_status`
- `verified_by`
- `verified_at`

---

## Import Report Example

```yaml
import_report:
  source_type: feishu_base
  bases_processed:
    - 华南旅游知识库

  tables_processed:
    - 目的地
    - 玩法
    - 酒店资源
    - 风险预警

  row_counts:
    destinations_raw: 12
    destinations_clean: 12
    pois_raw: 180
    pois_clean: 176
    experiences_raw: 95
    experiences_clean: 94
    resources_raw: 48
    resources_clean: 48
    risk_warnings_raw: 26
    risk_warnings_clean: 26

  warnings:
    - "3 experience records had non-standard validation_status values; mapped to pending"
    - "5 risk rows lacked related_ref; treated as city-level risks"

  dropped_rows:
    - table: 玩法
      row: 27
      reason: city missing
```

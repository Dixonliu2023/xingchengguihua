# Excel Adapter Spec

## Purpose

This adapter converts client-provided CSV or Excel files into the canonical
schema used by `travel-product-creator`.

Supported source files in the current client example:

- `目的地表.csv`
- `玩法表.csv`
- `资源表 1_完整补齐版.csv`
- `风险表.csv`

---

## Adapter Responsibilities

The Excel adapter should:

1. Read raw CSV or Excel files
2. Normalize fields and formats
3. Map records into canonical objects
4. Emit an import report

The adapter should not:

- design products
- validate routes with maps
- generate sales copy
- write data back to any knowledge base

---

## Supported Input Forms

### Multi-file mode

- multiple `.csv` files in one folder
- filename-based table detection

### Workbook mode

- one `.xlsx` file
- one business table per sheet

Recommended detection rules:

```yaml
file_detection:
  pois:
    aliases: ["目的地表", "景点表", "poi表"]

  experiences:
    aliases: ["玩法表", "体验表", "活动表"]

  resources:
    aliases: ["资源表", "供应商表", "酒店表", "餐厅表"]

  risk_warnings:
    aliases: ["风险表", "风险预警表"]
```

---

## Output Contract

```yaml
excel_adapter_output:
  canonical_data:
    destinations: []
    pois: []
    experiences: []
    resources: []
    risk_warnings: []
    rate_cards: []

  import_report:
    files_processed: []
    row_counts: {}
    warnings: []
    errors: []
    dropped_rows: []
```

`rate_cards` is reserved for future pricing support and may remain empty in the
first implementation.

---

## Global Cleaning Rules

```yaml
global_cleaning_rules:
  trim_whitespace: true
  normalize_fullwidth_punctuation: true
  empty_as_null: true

  array_splitters:
    - "，"
    - "、"
    - ","
    - "/"
    - ";"
    - "；"

  date_normalization:
    target_format: "YYYY-MM-DD"

  enum_normalization:
    severity:
      低: low
      中: medium
      高: high

    validation_status:
      pending: pending
      verified: verified
      rejected: rejected
      待验证: pending
      已验证: verified
      已拒绝: rejected
```

Additional rules:

- phone numbers stay as strings
- price text is preserved in `price_notes`
- unsafe numeric parsing falls back to warnings rather than destructive coercion

---

## Table-Level Mapping

## 1. `目的地表.csv` -> `pois`

```yaml
source_file: 目的地表.csv
target_object: pois

field_mapping:
  province: province
  city: city
  region: region
  name: poi_name
  description: poi_description
  tags: poi_tags
  best_season: best_season
  avg_days: visit_duration_text
  avg_budget: price_notes
```

Rules:

- generate `poi_id` as `poi_<city_pinyin>_<poi_name_pinyin>`
- split `poi_tags` into arrays
- keep `visit_duration_text` as raw text
- attempt to derive `visit_duration_hours`
- preserve `price_notes`
- attempt to derive `price_min` and `price_max`

---

## 2. `玩法表.csv` -> `experiences`

```yaml
source_file: 玩法表.csv
target_object: experiences

field_mapping:
  city: city
  title: experience_title
  category: experience_category_raw
  description: experience_description
  duration_hours: duration_hours
  suitable_for: suitable_for
  emotional_value: emotional_value
  highlights: highlights
  risk_warning: embedded_risk_warning
  source_file: source_file
  validation_status: validation_status
```

Rules:

- generate `experience_id` as `exp_<city_pinyin>_<title_pinyin>`
- split category, audience, and emotion fields into arrays
- coerce `duration_hours` into numeric form
- convert malformed values like `1 0` into `10`
- unresolved duration parsing should produce warnings

---

## 3. `资源表 1_完整补齐版.csv` -> `resources`

```yaml
source_file: 资源表 1_完整补齐版.csv
target_object: resources

field_mapping:
  city: city
  resource_type: resource_type
  name: resource_name
  contact_person: contact_person
  contact_phone: contact_phone
  cooperation_status: cooperation_status
  price_notes: price_notes
  suitable_for: suitable_for
  last_verified: verification_date
  star_rating: star_rating
```

Rules:

- generate `resource_id` as
  `res_<city_pinyin>_<resource_type_pinyin>_<resource_name_pinyin>`
- split `suitable_for` into arrays
- normalize `verification_date`
- preserve `price_notes`
- attempt to derive `price_min`

---

## 4. `风险表.csv` -> `risk_warnings`

```yaml
source_file: 风险表.csv
target_object: risk_warnings

field_mapping:
  city: city
  related_id: related_ref
  risk_type: risk_type
  risk_description: risk_description
  severity: severity
  mitigation_plan: mitigation_plan
  alternative: alternative_plan
```

Rules:

- generate `risk_id` as `risk_<city_pinyin>_<risk_type_pinyin>_<row_number>`
- map `severity` into `low|medium|high`
- if `related_ref` is missing, set `related_type = unknown`
- first implementation should treat these as city-level risks by default

---

## Derived Object: `destinations`

`destinations` should be derived, not directly imported, when the client only
provides POI-, experience-, resource-, and risk-level files.

```yaml
destinations_derived_from:
  - pois
  - experiences
  - resources
  - risk_warnings
```

Suggested derivation logic:

- group by `city`
- take `province` and `region` primarily from `pois`
- merge tags from POIs and experiences
- merge best seasons from POIs
- count linked POIs, experiences, resources, and risks
- generate `destination_id = dest_<city_pinyin>`

---

## Error Handling

### Missing values

- missing required fields: drop row and record in `dropped_rows`
- missing optional fields: set to null

### Dedupe keys

```yaml
dedupe_keys:
  pois: [city, poi_name]
  experiences: [city, experience_title]
  resources: [city, resource_type, resource_name]
  risk_warnings: [city, risk_type, risk_description]
```

### Malformed fields

- normalize text durations
- preserve multiline price text
- keep unsupported values as raw text with warnings

---

## Import Report Example

```yaml
import_report:
  files_processed:
    - 目的地表.csv
    - 玩法表.csv
    - 资源表 1_完整补齐版.csv
    - 风险表.csv

  row_counts:
    pois_raw: 120
    pois_clean: 118
    experiences_raw: 96
    experiences_clean: 95
    resources_raw: 65
    resources_clean: 65
    risk_warnings_raw: 22
    risk_warnings_clean: 22
    destinations_derived: 8

  warnings:
    - "玩法表.csv row 13 duration_hours='1 0' normalized to 10"
    - "风险表.csv has rows without related_ref; treated as city-level risks"

  dropped_rows:
    - file: 玩法表.csv
      row: 48
      reason: city missing
```

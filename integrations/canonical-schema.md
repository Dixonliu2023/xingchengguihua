# Canonical Schema

## Purpose

This schema defines the normalized data contract for `travel-product-creator`.
All knowledge sources must be converted into these canonical objects before
product design, route validation, document generation, or future pricing logic.

Supported source types:

- `excel_csv`
- `feishu_base`
- `mixed`

---

## Canonical Objects

- `destinations`
- `pois`
- `experiences`
- `resources`
- `risk_warnings`
- `rate_cards`

`rate_cards` is reserved for future pricing logic. It should be modeled now
even if pricing calculation is not yet fully implemented.

---

## 1. destinations

City- or region-level summary object used for positioning and route selection.

```yaml
destinations:
  primary_key: destination_id
  grain: one row per city_or_region
  required_fields:
    - destination_id
    - province
    - city
  fields:
    destination_id:
      type: string
      example: dest_gz

    province:
      type: string
      example: 广东

    city:
      type: string
      example: 广州

    region:
      type: string
      nullable: true

    destination_tags:
      type: string[]
      nullable: true

    best_season:
      type: string[]
      nullable: true

    suggested_stay:
      type: string
      nullable: true

    budget_band:
      type: string
      nullable: true

    tourism_index:
      type: number
      nullable: true

    poi_count:
      type: number
      nullable: true

    experience_count:
      type: number
      nullable: true

    resource_count:
      type: number
      nullable: true

    risk_count:
      type: number
      nullable: true

    source_type:
      type: string
      enum: [excel_csv, feishu_base, mixed]
```

---

## 2. pois

Point-of-interest object used for route drafting and map validation.

```yaml
pois:
  primary_key: poi_id
  grain: one row per poi
  required_fields:
    - poi_id
    - city
    - poi_name
  fields:
    poi_id:
      type: string
      example: poi_gz_guangzhouta

    destination_id:
      type: string
      nullable: true

    province:
      type: string
      nullable: true

    city:
      type: string

    region:
      type: string
      nullable: true

    poi_name:
      type: string

    poi_description:
      type: string
      nullable: true

    poi_tags:
      type: string[]
      nullable: true

    best_season:
      type: string[]
      nullable: true

    visit_duration_text:
      type: string
      nullable: true

    visit_duration_hours:
      type: number
      nullable: true

    price_notes:
      type: string
      nullable: true

    price_min:
      type: number
      nullable: true

    price_max:
      type: number
      nullable: true

    longitude:
      type: number
      nullable: true

    latitude:
      type: number
      nullable: true

    source_type:
      type: string
      enum: [excel_csv, feishu_base, mixed]

    source_file:
      type: string
      nullable: true
```

---

## 3. experiences

Experience-level object used for differentiation and experience design.

```yaml
experiences:
  primary_key: experience_id
  grain: one row per experience
  required_fields:
    - experience_id
    - city
    - experience_title
  fields:
    experience_id:
      type: string
      example: exp_gz_tower_nightview

    destination_id:
      type: string
      nullable: true

    city:
      type: string

    related_poi_id:
      type: string
      nullable: true

    experience_title:
      type: string

    experience_category_raw:
      type: string[]
      nullable: true

    experience_category_std:
      type: string[]
      nullable: true

    experience_description:
      type: string
      nullable: true

    duration_hours:
      type: number
      nullable: true

    suitable_for:
      type: string[]
      nullable: true

    emotional_value:
      type: string[]
      nullable: true

    highlights:
      type: string
      nullable: true

    wow_tags:
      type: string[]
      nullable: true

    embedded_risk_warning:
      type: string
      nullable: true

    source_file:
      type: string
      nullable: true

    validation_status:
      type: string
      enum: [pending, verified, rejected]
      default: pending

    source_type:
      type: string
      enum: [excel_csv, feishu_base, mixed]
```

---

## 4. resources

Operational resource directory for hotels, restaurants, vehicles, guides, and
other supply-side entities.

```yaml
resources:
  primary_key: resource_id
  grain: one row per resource
  required_fields:
    - resource_id
    - city
    - resource_type
    - resource_name
  fields:
    resource_id:
      type: string
      example: res_gz_hotel_whiteswan

    destination_id:
      type: string
      nullable: true

    city:
      type: string

    resource_type:
      type: string
      example: 酒店

    resource_subtype:
      type: string
      nullable: true

    resource_name:
      type: string

    contact_person:
      type: string
      nullable: true

    contact_phone:
      type: string
      nullable: true

    cooperation_status:
      type: string
      nullable: true

    price_notes:
      type: string
      nullable: true

    price_min:
      type: number
      nullable: true

    price_max:
      type: number
      nullable: true

    suitable_for:
      type: string[]
      nullable: true

    star_rating:
      type: string
      nullable: true

    verification_date:
      type: string
      nullable: true

    source_type:
      type: string
      enum: [excel_csv, feishu_base, mixed]
```

---

## 5. risk_warnings

Risk object used for route feasibility, product notes, and sales caution.

```yaml
risk_warnings:
  primary_key: risk_id
  grain: one row per risk
  required_fields:
    - risk_id
    - city
    - risk_type
    - risk_description
  fields:
    risk_id:
      type: string
      example: risk_gz_weather_001

    destination_id:
      type: string
      nullable: true

    city:
      type: string

    related_ref:
      type: string
      nullable: true

    related_type:
      type: string
      nullable: true
      enum: [destination, poi, experience, unknown]
      default: unknown

    risk_type:
      type: string

    risk_description:
      type: string

    severity:
      type: string
      enum: [low, medium, high]

    mitigation_plan:
      type: string
      nullable: true

    alternative_plan:
      type: string
      nullable: true

    source_type:
      type: string
      enum: [excel_csv, feishu_base, mixed]
```

---

## 6. rate_cards

Reserved pricing object for future quote and cost calculation features.

```yaml
rate_cards:
  primary_key: rate_id
  grain: one row per price rule
  required_fields:
    - rate_id
    - city
    - item_type
    - item_name
    - pricing_unit
    - base_price
  fields:
    rate_id:
      type: string
      example: rate_gz_tower_adult

    city:
      type: string

    item_type:
      type: string
      enum: [ticket, activity, hotel, transport, meal, guide, insurance, misc]

    item_name:
      type: string

    related_ref:
      type: string
      nullable: true

    pricing_unit:
      type: string
      enum:
        - per_person
        - per_room_night
        - per_vehicle_day
        - per_vehicle_trip
        - per_table
        - per_group
        - per_ticket

    base_price:
      type: number

    currency:
      type: string
      default: CNY

    traveler_type:
      type: string
      nullable: true

    room_type:
      type: string
      nullable: true

    vehicle_type:
      type: string
      nullable: true

    seat_count:
      type: number
      nullable: true

    season_type:
      type: string
      nullable: true

    valid_from:
      type: string
      nullable: true

    valid_to:
      type: string
      nullable: true

    cooperation_price:
      type: number
      nullable: true

    public_price:
      type: number
      nullable: true

    notes:
      type: string
      nullable: true

    last_verified:
      type: string
      nullable: true
```

---

## Common Rules

```yaml
common_rules:
  city:
    normalize: trim + standard city naming

  arrays:
    split_by: ["，", "、", ",", "/", ";", "；"]
    trim_items: true
    drop_empty: true

  dates:
    normalize_to: YYYY-MM-DD

  enums:
    validation_status:
      allowed: [pending, verified, rejected]

    severity:
      mapping:
        低: low
        中: medium
        高: high

  source_type:
    allowed: [excel_csv, feishu_base, mixed]
```

# Field Mapping Examples

## Purpose

This document records real examples from the current client CSV files and shows
how they map into canonical schema objects.

Source files:

- `目的地表.csv`
- `玩法表.csv`
- `资源表 1_完整补齐版.csv`
- `风险表.csv`

---

## 1. `目的地表.csv` should map to `pois`

This file is named like a destination table, but its row grain is clearly POI.

### Example 1

Input row:

```csv
province,city,region,name,description,tags,best_season,avg_days,avg_budget
广东,广州,珠三角核心区,越秀公园,广州市中心的大型综合性公园，有五羊石像等景点,城市公园，历史遗迹,春秋,2-3 小时,免费
```

Mapped output:

```yaml
poi_id: poi_gz_yuexiugongyuan
province: 广东
city: 广州
region: 珠三角核心区
poi_name: 越秀公园
poi_description: 广州市中心的大型综合性公园，有五羊石像等景点
poi_tags: ["城市公园", "历史遗迹"]
best_season: ["春秋"]
visit_duration_text: 2-3 小时
visit_duration_hours: 2.5
price_notes: 免费
price_min: 0
source_type: excel_csv
source_file: 目的地表.csv
```

### Example 2

Input row:

```csv
广东,广州,珠三角核心区,广州塔,中国第一高塔，城市地标，俯瞰广州全景,现代都市,全年,2-3 小时,150元起
```

Mapped output:

```yaml
poi_id: poi_gz_guangzhouta
province: 广东
city: 广州
region: 珠三角核心区
poi_name: 广州塔
poi_description: 中国第一高塔，城市地标，俯瞰广州全景
poi_tags: ["现代都市"]
best_season: ["全年"]
visit_duration_text: 2-3 小时
visit_duration_hours: 2.5
price_notes: 150元起
price_min: 150
source_type: excel_csv
source_file: 目的地表.csv
```

### Important note

The original fields `avg_days` and `avg_budget` should not be treated literally
in this source file:

- `avg_days` behaves like POI visit duration
- `avg_budget` behaves like POI ticket or price note

---

## 2. Multiline POI price notes

Some rows contain tiered ticket descriptions.

Input fragment:

```text
1. 433 米白云星空观光票：150 元
2. 450 米户外平台观光票：228 元
3. 460 米摩天轮套票：298 元
```

Mapped output:

```yaml
price_notes: |
  1. 433米白云星空观光票：150元
  2. 450米户外平台观光票：228元
  3. 460米摩天轮套票：298元
price_min: 150
price_max: 298
```

Rule:

- preserve raw pricing text
- extract min and max numeric values when safe

---

## 3. `玩法表.csv` -> `experiences`

### Example 1

Input row:

```csv
city,title,category,description,duration_hours,suitable_for,emotional_value,highlights,risk_warning,source_file,validation_status
广州,游览越秀公园,城市公园，历史遗迹,广州市中心的大型综合性公园，有五羊石像等景点,2,情侣，朋友，家庭,放松身心，开阔视野,广州市中心的大型综合性公园，有五羊石像等景点,,我是驴友-广州旅游攻略.pdf,pending
```

Mapped output:

```yaml
experience_id: exp_gz_visit_yuexiugongyuan
city: 广州
experience_title: 游览越秀公园
experience_category_raw: ["城市公园", "历史遗迹"]
experience_description: 广州市中心的大型综合性公园，有五羊石像等景点
duration_hours: 2
suitable_for: ["情侣", "朋友", "家庭"]
emotional_value: ["放松身心", "开阔视野"]
highlights: 广州市中心的大型综合性公园，有五羊石像等景点
embedded_risk_warning: null
source_file: 我是驴友-广州旅游攻略.pdf
validation_status: pending
source_type: excel_csv
```

### Example 2

Input row:

```csv
广州,游览长隆野生动物世界,动物园，亲子,大型野生动物园，可近距离接触动物,6,情侣，朋友，家庭,放松身心，开阔视野,大型野生动物园，可近距离接触动物,,我是驴友-广州旅游攻略.pdf,pending
```

Mapped output:

```yaml
experience_id: exp_gz_visit_changlongzoo
city: 广州
experience_title: 游览长隆野生动物世界
experience_category_raw: ["动物园", "亲子"]
experience_description: 大型野生动物园，可近距离接触动物
duration_hours: 6
suitable_for: ["情侣", "朋友", "家庭"]
emotional_value: ["放松身心", "开阔视野"]
highlights: 大型野生动物园，可近距离接触动物
validation_status: pending
source_type: excel_csv
```

---

## 4. Dirty duration example in `玩法表.csv`

Input row fragment:

```csv
深圳,游览东部华侨城,主题乐园，度假,大型综合旅游度假区,1 0,情侣，朋友，家庭,放松身心，开阔视野,大型综合旅游度假区,,我是驴友-深圳旅游攻略.pdf,pending
```

Mapped output:

```yaml
duration_hours_raw: "1 0"
duration_hours: 10
cleaning_warning: "duration_hours normalized from '1 0' to 10"
```

Rule:

- remove accidental spacing in numeric duration fields
- if parsing still fails, set `duration_hours: null` and emit warning

---

## 5. `资源表 1_完整补齐版.csv` -> `resources`

### Example 1

Input row:

```csv
city,resource_type,name,contact_person,contact_phone,cooperation_status,price_notes,suitable_for,last_verified,star_rating
广州,酒店,白天鹅宾馆,销售部,020-81886968,可合作,豪华房 ¥2680 起，玉堂春暖餐厅人均 ¥500,高端商务、政务接待、粤菜体验,2026/5/12,五星
```

Mapped output:

```yaml
resource_id: res_gz_hotel_whiteswan
city: 广州
resource_type: 酒店
resource_name: 白天鹅宾馆
contact_person: 销售部
contact_phone: 020-81886968
cooperation_status: 可合作
price_notes: 豪华房 ¥2680 起，玉堂春暖餐厅人均 ¥500
price_min: 2680
suitable_for: ["高端商务", "政务接待", "粤菜体验"]
verification_date: 2026-05-12
star_rating: 五星
source_type: excel_csv
source_file: 资源表 1_完整补齐版.csv
```

### Example 2

Input row:

```csv
深圳,酒店,深圳机场凯悦嘉轩酒店,销售经理,0755-23458888,可合作,标准间 ¥650 起，含双早，机场接送,商务出行、机场中转,2026/5/12,四星
```

Mapped output:

```yaml
resource_id: res_sz_hotel_hyattplace_airport
city: 深圳
resource_type: 酒店
resource_name: 深圳机场凯悦嘉轩酒店
contact_person: 销售经理
contact_phone: 0755-23458888
cooperation_status: 可合作
price_notes: 标准间 ¥650 起，含双早，机场接送
price_min: 650
suitable_for: ["商务出行", "机场中转"]
verification_date: 2026-05-12
star_rating: 四星
source_type: excel_csv
```

---

## 6. `风险表.csv` -> `risk_warnings`

### Example 1

Input row:

```csv
city,related_id,risk_type,risk_description,severity,mitigation_plan,alternative
广州,GZ-002,景区预约风险,热门景点需提前 3-5 天预约，现场无票率高,中,提前 7 天预约,备选开放式街区
```

Mapped output:

```yaml
risk_id: risk_gz_booking_002
city: 广州
related_ref: GZ-002
related_type: unknown
risk_type: 景区预约风险
risk_description: 热门景点需提前 3-5 天预约，现场无票率高
severity: medium
mitigation_plan: 提前 7 天预约
alternative_plan: 备选开放式街区
source_type: excel_csv
source_file: 风险表.csv
```

### Example 2

Input row with missing related id:

```csv
深圳,,交通,高峰期交通拥堵,中,错峰出行，使用地铁,提前规划路线
```

Mapped output:

```yaml
risk_id: risk_sz_traffic_001
city: 深圳
related_ref: null
related_type: unknown
risk_type: 交通
risk_description: 高峰期交通拥堵
severity: medium
mitigation_plan: 错峰出行，使用地铁
alternative_plan: 提前规划路线
source_type: excel_csv
source_file: 风险表.csv
```

Rule:

- rows without `related_ref` are valid
- first implementation should treat them as city-level risks

---

## 7. Derived `destinations` example

Input signals:

- `pois.city = 广州`
- `experiences.city = 广州`
- `resources.city = 广州`
- `risk_warnings.city = 广州`

Derived output:

```yaml
destination_id: dest_gz
province: 广东
city: 广州
region: 珠三角核心区
destination_tags:
  - 城市公园
  - 历史遗迹
  - 博物馆
  - 亲子
  - 美食
best_season:
  - 春秋
  - 全年
poi_count: 4
experience_count: 10
resource_count: 7
risk_count: 5
source_type: excel_csv
```

Rule:

- derive `destination_tags` from POI tags and experience categories
- merge `best_season` from POIs
- prefer POI-derived province and region values

---

## 8. Future pricing examples

The current client files do not yet provide a dedicated pricing table, but
future pricing rows should map into `rate_cards`.

Example transport row:

```yaml
rate_id: rate_gz_mpv7_day
city: 广州
item_type: transport
item_name: 7座MPV包车
pricing_unit: per_vehicle_day
base_price: 1200
vehicle_type: 7座MPV
seat_count: 7
currency: CNY
```

Example hotel rate row:

```yaml
rate_id: rate_gz_hotel_standard_room
city: 广州
item_type: hotel
item_name: 标准间
pricing_unit: per_room_night
base_price: 580
room_type: twin
currency: CNY
```

Example ticket row:

```yaml
rate_id: rate_gz_tower_adult
city: 广州
item_type: ticket
item_name: 广州塔433米观光票
pricing_unit: per_ticket
base_price: 150
traveler_type: adult
currency: CNY
```

---

## 9. Current Data Quality Summary

This client package is usable, but several issues should be expected:

1. `目的地表.csv` is misnamed and should be treated as POI data
2. `玩法表.csv` uses broad emotional tags with limited discrimination power
3. `玩法表.csv` contains malformed duration values such as `1 0`
4. `风险表.csv` uses unstable `related_id` values
5. `资源表 1_完整补齐版.csv` is currently hotel-heavy and not yet a complete
   supply-side inventory

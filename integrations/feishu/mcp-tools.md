# 飞书多维表格MCP工具定义

## 概述

本文档定义与飞书多维表格集成的MCP工具，用于访问飞书知识库中的旅游数据。

## 认证配置

```yaml
# feishu-config.yaml
feishu:
  app_id: "cli_xxxxxxxxx"
  app_secret: "xxxxxxxxxxxx"
  app_token: "app_xxxxxxxxxxxxx"
  
# 环境变量
# FEISHU_APP_ID
# FEISHU_APP_SECRET  
# FEISHU_APP_TOKEN
```

## 工具列表

### 1. search_destinations

**描述：** 搜索目的地基础信息

**输入参数：**
```json
{
  "city": "string (optional)",
  "region": "string (optional)",
  "tags": ["string"] (optional)
}
```

**输出：**
```json
{
  "destinations": [
    {
      "destination_id": "string",
      "city": "string",
      "region": "string",
      "province": "string",
      "description": "string",
      "tags": ["string"],
      "avg_rating": "number",
      "is_popular": "boolean"
    }
  ],
  "total": "number"
}
```

---

### 2. search_special_experiences

**描述：** 搜索特别玩法库

**输入参数：**
```json
{
  "city": "string (optional)",
  "category": "string (optional)",
  "budgetMin": "number (optional)",
  "budgetMax": "number (optional)",
  "difficulty": "string (optional)",
  "is_rare": "boolean (optional)"
}
```

**输出：**
```json
{
  "experiences": [
    {
      "experience_id": "string",
      "title": "string",
      "city": "string",
      "category": "string",
      "sub_categories": ["string"],
      "description": "string",
      "duration": "number",
      "price": "number",
      "difficulty": "string",
      "suitable_for": ["string"],
      "emotional_value": ["string"],
      "is_rare": "boolean",
      "social_score": "number",
      "wow_moment_tags": ["string"],
      "highlights": "string",
      "requirements": "string",
      "avg_rating": "number",
      "risk_warning": "string (optional)"
    }
  ],
  "total": "number"
}
```

---

### 3. search_by_categories

**描述：** 按主题分类搜索体验

**输入参数：**
```json
{
  "categories": ["string"],
  "city": "string (optional)",
  "emotional_value": ["string"] (optional),
  "suitable_for": ["string"] (optional)
}
```

**输出：**
```json
{
  "experiences": [
    {
      "experience_id": "string",
      "title": "string",
      "category": "string",
      "emotional_value": ["string"],
      "suitable_for": ["string"],
      "city": "string",
      "duration": "number",
      "price": "number",
      "social_score": "number",
      "is_rare": "boolean"
    }
  ],
  "category_match": {
    "requested": ["string"],
    "matched": ["string"]
  }
}
```

---

### 4. sync_retrieve_experiences

**描述：** 景点筛选阶段同步检索飞书特别玩法（核心创新）

**输入参数：**
```json
{
  "city": "string",
  "categories": ["string"],
  "populationType": "string",
  "durationRange": {
    "min": "number",
    "max": "number"
  },
  "emotional_value": ["string"] (optional)
}
```

**输出：**
```json
{
  "fusion_result": {
    "total": "number",
    "standard_poi_count": "number",
    "special_experience_count": "number"
  },
  "standard_pois": [
    {
      "name": "string",
      "type": "scenic | restaurant | hotel",
      "rating": "number",
      "visit_duration": "number"
    }
  ],
  "special_experiences": [
    {
      "id": "string",
      "title": "string",
      "category": "string",
      "duration": "number",
      "price": "number",
      "emotional_value": ["string"],
      "suitable_for": ["string"],
      "highlights": "string",
      "is_rare": "boolean",
      "feishu_source": true,
      "social_score": "number",
      "wow_moment_tags": ["string"],
      "risk_warning": "string"
    }
  ],
  "match_scores": {
    "overall": "number",
    "category_match": "number",
    "population_match": "number",
    "duration_match": "number"
  }
}
```

**核心逻辑：**
1. 同时查询标准景点库和飞书特别玩法库
2. 按主题匹配度和人群适合度计算综合评分
3. 返回融合后的候选清单（带来源标识）
4. 飞书来源的玩法标记 `feishu_source: true`

---

### 5. get_destination_elements

**描述：** 获取目的地元素挖掘（Product Designer资源解构用）

**输入参数：**
```json
{
  "city": "string"
}
```

**输出：**
```json
{
  "destination_id": "string",
  "city": "string",
  "elements": {
    "pastoral": ["string"],
    "agricultural": ["string"],
    "commercial": ["string"],
    "handcraft": ["string"],
    "religious": ["string"],
    "historical": ["string"]
  },
  "hidden_beauty": [
    {
      "description": "string",
      "type": "string",
      "accessibility": "string"
    }
  ],
  "stories": [
    {
      "title": "string",
      "content": "string",
      "theme": "string"
    }
  ]
}
```

---

### 6. write_back_to_feishu

**描述：** 知识库回写（产品生成的新亮点存入飞书）

**输入参数：**
```json
{
  "experienceData": {
    "title": "string",
    "city": "string",
    "category": "string",
    "sub_categories": ["string"],
    "description": "string",
    "duration": "number",
    "price": "number",
    "difficulty": "string",
    "suitable_for": ["string"],
    "emotional_value": ["string"],
    "highlights": "string",
    "requirements": "string",
    "is_rare": "boolean"
  },
  "source_info": {
    "source": "product_generation",
    "original_product_id": "string",
    "generated_at": "string"
  },
  "dedup_check": true
}
```

**输出：**
```json
{
  "status": "success | duplicate | error",
  "record_id": "string",
  "is_new": "boolean",
  "dedup_result": {
    "checked": "boolean",
    "similar_records": [
      {
        "id": "string",
        "title": "string",
        "similarity": "number"
      }
    ]
  },
  "validation_status": "pending_verification"
}
```

---

### 7. get_risk_warnings

**描述：** 获取特定玩法的风险预警

**输入参数：**
```json
{
  "experience_id": "string (optional)",
  "city": "string (optional)",
  "category": "string (optional)"
}
```

**输出：**
```json
{
  "risk_warnings": [
    {
      "experience_id": "string",
      "experience_title": "string",
      "risk_type": "string",
      "risk_description": "string",
      "severity": "low | medium | high",
      "mitigation_plan": "string",
      "alternative": "string (optional)"
    }
  ]
}
```

---

### 8. match_customer_requirements

**描述：** 根据客户需求匹配玩法

**输入参数：**
```json
{
  "destinations": ["string"],
  "categories": ["string"],
  "budget_min": "number",
  "budget_max": "number",
  "duration_min": "number",
  "duration_max": "number",
  "suitable_for": ["string"],
  "emotional_value": ["string"]
}
```

**输出：**
```json
{
  "matches": [
    {
      "experience_id": "string",
      "title": "string",
      "city": "string",
      "category": "string",
      "match_score": "number",
      "score_breakdown": {
        "destination": "number",
        "category": "number",
        "budget": "number",
        "duration": "number",
        "population": "number",
        "emotion": "number"
      },
      "price": "number",
      "duration": "number",
      "emotional_value": ["string"],
      "suitable_for": ["string"],
      "is_rare": "boolean",
      "social_score": "number"
    }
  ],
  "total_matches": "number",
  "top_recommendations": ["string"]
}
```

---

## 数据模型

### destinations 表

| 字段名 | 类型 | 说明 |
|-------|------|------|
| destination_id | text | 目的地ID |
| city | text | 城市 |
| region | text | 区域 |
| province | text | 省份 |
| description | text | 描述 |
| tags | multiSelect | 标签 |
| avg_rating | rating | 平均评分 |
| is_popular | checkbox | 是否热门 |

### special_experiences 表

| 字段名 | 类型 | 说明 |
|-------|------|------|
| experience_id | text | 玩法ID |
| title | text | 标题 |
| city | text | 城市 |
| category | select | 主题分类 |
| sub_categories | multiSelect | 子分类 |
| description | text | 详细描述 |
| duration | number | 时长（小时） |
| price | number | 价格 |
| difficulty | select | 难度等级 |
| suitable_for | multiSelect | 适合人群 |
| emotional_value | multiSelect | 情绪价值 |
| is_rare | checkbox | 是否稀缺 |
| social_score | rating | 社交评分 |
| wow_moment_tags | multiSelect | 哇点标签 |
| highlights | text | 亮点 |
| requirements | text | 参与要求 |
| avg_rating | rating | 平均评分 |
| risk_warning | text | 风险预警 |
| source | select | 来源 |
| validation_status | select | 验证状态 |

---

## 错误处理

| 错误码 | 说明 | 处理方式 |
|-------|------|---------|
| 99991668 | Token过期 | 重新获取token后重试 |
| 99991663 | 权限不足 | 检查应用权限配置 |
| 99991701 | 表格不存在 | 检查app_token和table_id |
| 99991702 | 记录不存在 | 返回空结果 |

---

## 缓存策略

**缓存TTL：** 5分钟

**缓存键：** `feishu_{table}_{filter_hash}`

**优化建议：**
- 热门目的地数据可设置更长缓存（30分钟）
- 特别玩法数据建议实时查询（确保准确性）

---

*文档版本：v1.0*
*创建日期：2026-03-10*
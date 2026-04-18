# 高德MCP工具定义

## 概述

本文档定义高德地图MCP工具，用于验证景点通达性和路径优化。

## 认证配置

高德MCP使用高德开放平台Web服务API，需要配置API Key。

```yaml
# 高德配置（通过环境变量）
# AMAP_API_KEY - 高德Web服务API Key
# AMAP_API_SECURITY - 高德签名密钥（可选）
```

## 工具列表

### 1. check_route_accessibility

**描述：** 验证两个景点之间的通达性

**输入参数：**
```json
{
  "origin": {
    "name": "string",
    "lng": "number (optional)",
    "lat": "number (optional)"
  },
  "destination": {
    "name": "string", 
    "lng": "number (optional)",
    "lat": "number (optional)"
  },
  "strategy": "drive | transit | walking"
}
```

**输出：**
```json
{
  "origin": "string",
  "destination": "string",
  "distance_km": "number",
  "duration_minutes": "number",
  "strategy": "string",
  "status": "reachable | unreachable | needs_adjustment",
  "recommendation": "string",
  "traffic_condition": "string (optional)",
  "alternative_route": {
    "distance_km": "number",
    "duration_minutes": "number"
  }
}
```

---

### 2. validate_route_logical

**描述：** 验证整条线路的逻辑性（不走回头路）

**输入参数：**
```json
{
  "pois": [
    {
      "name": "string",
      "lng": "number (optional)",
      "lat": "number (optional)",
      "day": "number"
    }
  ],
  "start_point": {
    "name": "string",
    "lng": "number",
    "lat": "number"
  },
  "end_point": {
    "name": "string", 
    "lng": "number,
    "lat": "number
  }
}
```

**输出：**
```json
{
  "validation_result": {
    "is_logical": "boolean",
    "total_distance_km": "number",
    "total_duration_minutes": "number",
    "backtrack_segments": [
      {
        "from": "string",
        "to": "string",
        "distance_km": "number",
        "issue": "string"
      }
    ]
  },
  "daily_summary": [
    {
      "day": "number",
      "pois_count": "number",
      "distance_km": "number",
      "duration_minutes": "number",
      "is_efficient": "boolean"
    }
  ],
  "recommendations": [
    {
      "type": "optimization",
      "description": "string",
      "suggested_change": "string"
    }
  ]
}
```

---

### 3. optimize_route_order

**描述：** 优化景点游览顺序（减少绕路）

**输入参数：**
```json
{
  "pois": [
    {
      "name": "string",
      "lng": "number",
      "lat": "number,
      "visit_duration": "number",
      "must_visit": "boolean (optional)"
    }
  ],
  "start_point": {
    "name": "string",
    "lng": "number",
    "lat": "number"
  },
  "end_point": {
    "name": "string",
    "lng": "number",
    "lat": "number"
  }
}
```

**输出：**
```json
{
  "original_order": ["string"],
  "optimized_order": ["string"],
  "optimization_result": {
    "original_total_km": "number",
    "optimized_total_km": "number",
    "saved_km": "number",
    "saved_percentage": "number"
  },
  "daily_itinerary": [
    {
      "day": "number",
      "pois": ["string"],
      "total_distance_km": "number",
      "estimated_time": "string"
    }
  ]
}
```

---

### 4. get_nearby_pois

**描述：** 获取指定地点周边的景点

**输入参数：**
```json
{
  "location": {
    "lng": "number",
    "lat": "number
  },
  "radius_km": "number",
  "types": ["string"] (optional),
  "limit": "number"
}
```

**输出：**
```json
{
  "pois": [
    {
      "name": "string",
      "type": "string",
      "distance_km": "number",
      "rating": "number",
      "address": "string",
      "opening_hours": "string"
    }
  ],
  "total": "number"
}
```

---

### 5. calculate_route_cost

**描述：** 计算交通成本

**输入参数：**
```json
{
  "segments": [
    {
      "from": "string",
      "to": "string",
      "distance_km": "number",
      "transport_mode": "self_drive | bus | taxi | train"
    }
  ],
  "vehicle_type": "car | bus (optional)",
  "fuel_price": "number (optional)",
  "toll_cost": "number (optional)"
}
```

**输出：**
```json
{
  "segments": [
    {
      "from": "string",
      "to": "string",
      "distance_km": "number",
      "transport_mode": "string",
      "estimated_cost": "number"
    }
  ],
  "total_cost": "number",
  "cost_breakdown": {
    "fuel": "number",
    "toll": "number",
    "ticket": "number"
  }
}
```

---

## 验证规则

### 自驾验证规则

| 距离范围 | 状态 | 建议 |
|---------|------|------|
| ≤50km 或 ≤60分钟 | ✅ 可达 | 正常安排 |
| 50-80km 或 60-90分钟 | ⚠️ 需调整 | 建议优化顺序或替代方案 |
| >80km 或 >90分钟 | ❌ 不可达 | 必须调整或分拆 |

### 公共交通验证规则

| 换乘次数 | 状态 | 建议 |
|---------|------|------|
| 0-1次 | ✅ 可达 | 正常安排 |
| 2-3次 | ⚠️ 需调整 | 考虑包车或自驾 |
| >3次 | ❌ 不可达 | 必须调整 |

### "不走回头路"规则

1. 检查是否有景点被重复经过
2. 检查顺序是否符合地理方向（由北到南/由近到远）
3. 检查是否形成闭环（终点可返回起点）

---

## 输出格式

### 景点通达性验证表

```
### 景点通达性验证
| 起点 | 终点 | 距离 | 车程 | 建议交通 | 状态 | 风险预警 |
|------|------|------|------|----------|------|----------|
| 星海广场 | 老虎滩 | 8km | 20分钟 | 自驾 | ✅可达 | - |
| 老虎滩 | 金石滩 | 45km | 50分钟 | 自驾 | ✅可达 | 🟡旺季拥堵 |
```

### 走回头路检查

```
### 走回头路检查
□ Day1: 星海广场 → 老虎滩 → 金石滩
  状态：✅ 单向无回头

□ Day2: 金石滩 → 森林动物园 → 星海广场
  状态：⚠️ 有回头趋势，建议调整
```

---

## 错误处理

| 错误码 | 说明 | 处理方式 |
|-------|------|---------|
| 10001 | 认证失败 | 检查API Key |
| 10002 | 配额不足 | 联系高德续费 |
| 20001 | 起点/终点不存在 | 检查POI名称 |
| 20002 | 无规划结果 | 调整POI或交通方式 |

---

*文档版本：v1.0*
*创建日期：2026-03-10*
# 飞书数据模型设计

## 概述

本文档定义飞书多维表格的数据模型，用于存储旅游知识库数据。

## 表结构总览

| 表名 | 用途 | 核心字段 |
|------|------|---------|
| destinations | 目的地基础信息 | city, region, avg_rating, tags |
| special_experiences | 特别玩法库 | title, category, emotional_value, is_rare |
| experience_categories | 主题分类树 | category_id, name, parent_id |
| risk_warnings | 风险预警库 | experience_id, risk_type, mitigation_plan |
| new_highlights | 新亮点库（回写） | source, validation_status, feedback_score |
| product_logs | 产品生成日志 | product_id, route_type, generated_docs |

---

## 1. destinations - 目的地基础信息表

### 表结构

| 字段名 | 字段类型 | 说明 | 示例值 |
|-------|---------|------|-------|
| destination_id | text | 目的地ID | dest_bj_001 |
| city | text | 城市 | 北京 |
| region | text | 区域 | 华北 |
| province | text | 省份 | 北京市 |
| cover_image | attachment | 封面图 | [图片URL] |
| description | text | 描述 | 千年古都，现代与历史交融 |
| tags | multiSelect | 标签 | ["历史文化", "美食", "购物"] |
| avg_rating | rating | 平均评分 | 4.8 |
| is_popular | checkbox | 是否热门 | true |
| tourism_index | number | 旅游指数(0-100) | 95 |
| best_season | multiSelect | 最佳旅游季节 | ["春季", "秋季"] |
| avg_days | number | 推荐游玩天数 | 5 |
| avg_budget | number | 人均预算(元) | 3000 |

### 使用场景

- 路由判断：目的地热度评分
- 产品定位：目的地背景信息
- 资源解构：目的地元素挖掘

---

## 2. special_experiences - 特别玩法库（核心表）

### 表结构

| 字段名 | 字段类型 | 说明 | 示例值 |
|-------|---------|------|-------|
| experience_id | text | 玩法ID | exp_bj_001 |
| title | text | 标题 | 故宫深度导览 |
| destination_id | text | 目的地ID（关联） | dest_bj_001 |
| city | text | 城市 | 北京 |
| category | select | 主题分类 | 历史文化 |
| sub_categories | multiSelect | 子分类 | ["建筑", "皇家", "文物"] |
| description | text | 详细描述 | 专家讲解，深度探索... |
| duration | number | 时长（小时） | 4 |
| price | number | 价格 | 299 |
| difficulty | select | 难度等级 | 轻松 |
| min_participants | number | 最少人数 | 1 |
| max_participants | number | 最多人数 | 10 |
| suitable_for | multiSelect | 适合人群 | ["亲子", "情侣", "银发"] |
| emotional_value | multiSelect | 情绪价值 | ["惊喜", "怀旧", "探索"] |
| is_rare | checkbox | 是否稀缺玩法 | true |
| social_score | rating | 社交传播潜力(1-5) | 4.5 |
| wow_moment_tags | multiSelect | 哇点标签 | ["日出", "文化表演"] |
| highlights | text | 亮点 | - 穿越文华殿\n- 体验科举制度 |
| requirements | text | 参与要求 | 需要提前3天预约 |
| provider | text | 提供商 | 文化之旅 |
| contact_phone | phone | 联系电话 | 13800138000 |
| booking_url | url | 预订链接 | https://... |
| is_available | checkbox | 是否可预订 | true |
| risk_warning | text | 风险预警 | 旺季需提前预约 |
| images | attachment | 图片集 | [多张图片] |
| avg_rating | rating | 平均评分 | 4.8 |
| review_count | number | 评价数量 | 128 |
| created_time | datetime | 创建时间 | 2024-01-01T10:00:00 |
| updated_time | datetime | 更新时间 | 2024-03-10T15:30:00 |
| source | select | 来源 | 手动录入/产品生成回写 |
| validation_status | select | 验证状态 | 待验证/已验证/已拒绝 |
| is_featured | checkbox | 是否精选 | false |

### 使用场景

- 玩法注入：同步检索飞书特别玩法
- 产品设计：哇点设计、情绪价值植入
- 销售推介：差异化卖点提炼
- 知识库回写：新亮点自动录入

---

## 3. experience_categories - 主题分类表

### 表结构

| 字段名 | 字段类型 | 说明 | 示例值 |
|-------|---------|------|-------|
| category_id | text | 分类ID | cat_culture |
| name | text | 分类名称 | 历史文化 |
| parent_id | text | 父分类ID | null |
| icon | attachment | 图标 | [图标] |
| description | text | 描述 | 探索历史遗迹和文化传统 |
| sort_order | number | 排序 | 1 |
| is_active | checkbox | 是否启用 | true |

### 分类树结构

```
历史文化
├─ 建筑遗产
├─ 考古探索
└─ 非遗体验

自然风光
├─ 山地徒步
├─ 海滨度假
└─ 森林探险

美食体验
├─ 街头小吃
├─ 精致餐厅
└─ 烹饪课程

艺术文化
├─ 博物馆
├─ 艺术展览
└─ 表演艺术

都市体验
├─ 购物
├─ 夜生活
└─ 城市探索

户外运动
├─ 水上活动
├─ 极限运动
└─ 徒步登山

亲子活动
├─ 游乐园
├─ 动物园
└─ 寓教于乐
```

---

## 4. risk_warnings - 风险预警表

### 表结构

| 字段名 | 字段类型 | 说明 | 示例值 |
|-------|---------|------|-------|
| risk_id | text | 风险ID | risk_001 |
| experience_id | text | 玩法ID（关联） | exp_bj_001 |
| experience_title | text | 玩法标题 | 故宫深度导览 |
| city | text | 城市 | 北京 |
| risk_type | select | 风险类型 | 天气/预约/安全/交通 |
| risk_description | text | 风险描述 | 旺季可能限流 |
| severity | select | 严重程度 | low/medium/high |
| mitigation_plan | text | 应对方案 | 提前7天预约 |
| alternative | text | 替代方案 | 更换为淡季出行 |
| season_restrictions | text | 季节限制 | 雨季不建议 |
| last_verified | datetime | 最后验证时间 | 2024-03-01 |

---

## 5. new_highlights - 新亮点库（回写用）

### 表结构

| 字段名 | 字段类型 | 说明 | 示例值 |
|-------|---------|------|-------|
| highlight_id | text | 亮点ID | hl_001 |
| title | text | 亮点标题 | 私人露营看星星 |
| city | text | 城市 | 青海 |
| category | text | 分类 | 自然风光 |
| description | text | 描述 | ... |
| source | select | 来源 | product_generation/manual |
| original_product_id | text | 原产品ID | prod_001 |
| generated_at | datetime | 生成时间 | 2024-03-10 |
| validation_status | select | 验证状态 | 待验证/已验证/已拒绝 |
| feedback_score | rating | 客户反馈评分 | 4.5 |
| verified_by | text | 验证人 | 张三 |
| verified_at | datetime | 验证时间 | 2024-03-15 |
| is_published | checkbox | 是否发布 | false |

---

## 6. product_logs - 产品生成日志表

### 表结构

| 字段名 | 字段类型 | 说明 | 示例值 |
|-------|---------|------|-------|
| log_id | text | 日志ID | log_001 |
| product_id | text | 产品ID | prod_001 |
| product_name | text | 产品名称 | 大连3天2晚亲子游 |
| route_type | select | 工作流类型 | quick/deep |
| input_requirements | text | 输入需求(JSON) | {...} |
| feishu_experiences_used | number | 使用飞书玩法数 | 3 |
| documents_generated | multiSelect | 生成的文档 | ["简易行程表","销售推介"] |
| generation_time | number | 生成耗时(秒) | 180 |
| status | select | 状态 | success/error |
| error_message | text | 错误信息 | null |
| created_at | datetime | 创建时间 | 2024-03-10T10:00:00 |

---

## 数据联动关系

```
destinations (1) ──────< (N) special_experiences
                                              │
                                              ├─< (N) risk_warnings
                                              │
                                              └─> (1) experience_categories

special_experiences ──────> new_highlights (回写)
                                              │
product_logs (日志追踪) <─────────────────────┘
```

---

## 数据维护规范

### 新增数据

1. 手动录入：运营人员按模板填写
2. API回写：产品生成自动触发（需去重检查）
3. 导入：批量数据导入（CSV/Excel）

### 数据校验

1. 必填字段：`title`, `city`, `category`
2. 推荐字段：`emotional_value`, `suitable_for`, `highlights`
3. 评分字段：`avg_rating`, `social_score` 需≥1分

### 定期维护

| 维护项 | 频率 | 负责人 |
|--------|------|--------|
| 数据完整性检查 | 每周 | 运营 |
| 玩法评分更新 | 每月 | 运营 |
| 风险预警复核 | 每季度 | 计调 |
| 知识库清理 | 每半年 | 管理员 |

---

*文档版本：v1.0*
*创建日期：2026-03-10*
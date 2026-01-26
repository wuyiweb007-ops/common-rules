# 产品表单与打印功能规格说明

## 1. 概述

完善产品模块的新增/编辑表单，并增加批量打印功能。

## 2. 产品表单改造

### 2.1 表单字段布局

采用双列布局，每行两个字段：

| 左列 | 右列 |
|------|------|
| 产品名称 * | 规格型号 * |
| AI编码 (只读) | 产品编号 * |
| 产品图号 | 商品别名 |
| 商品分类 | 客户名称 * |
| 数量 | 审核 |
| 批准 | 外观 |
| 备注 | 状态 |

> `*` 表示必填字段

### 2.2 字段说明

| 字段 | 数据库字段 | 类型 | 必填 | 说明 |
|------|-----------|------|------|------|
| 产品名称 | product_name | input | 是 | |
| 规格型号 | product_spec | input | 是 | |
| AI编码 | ai_id | input | - | 只读，系统自动生成 |
| 产品编号 | product_code | input | 是 | |
| 产品图号 | product_drawing_no | input | 否 | |
| 商品别名 | product_alias | input | 否 | |
| 商品分类 | product_category | input | 否 | |
| 客户名称 | customer | input | 是 | |
| 数量 | quantity | input | 否 | |
| 审核 | review | input | 否 | |
| 批准 | approval | input | 否 | |
| 外观 | appearance | input | 否 | |
| 备注 | remark | input | 否 | |
| 状态 | status | radio | 否 | active/inactive |

### 2.3 AI编码生成规则

- 新增产品时，由后端自动生成
- 格式：`AI{company_id}{timestamp后6位}`
- 前端不传递ai_id，由后端生成
- 编辑时ai_id为只读

### 2.4 工序管理

- **新增产品**：保存产品后自动进入编辑模式，展示工序管理区域
- **编辑产品**：直接展示工序管理区域
- 工序列表字段：工序名称、规格型号(spec)、单价、排序、状态

## 3. 复制产品功能

### 3.1 交互流程

1. 产品列表操作栏增加"复制"按钮
2. 点击复制，打开新增对话框
3. 带入当前产品信息（不含id和ai_id）
4. 用户修改后保存，创建新产品
5. ai_id由系统自动生成

### 3.2 复制字段

复制以下字段：
- product_name
- product_spec
- product_code（需用户修改）
- product_drawing_no
- product_alias
- product_category
- customer
- quantity
- review
- approval
- appearance
- remark
- status
- processes（工序列表）

## 4. 打印功能

### 4.1 交互流程

1. 列表勾选产品（支持多选）
2. 点击"打印"按钮
3. 选择打印版式（A/B）
4. 打开打印预览
5. 调用浏览器打印

### 4.2 版式A - 纵向布局

- A4纸纵向
- 每页2个产品
- 上下排列

```
┌─────────────────────────┐
│    刹车线生产工序表       │
│ 产品名称: xxx  客户: xxx │
│ 产品编号: xxx  数量:     │
│ 产品规格: xxx  审核:     │
│ AI编号: xxx    批准:     │
│ 序号 | 工位 | 规格型号 | 外观 | 备注 │
│  1   | xxx  | xxx     |     |      │
│  2   | xxx  | xxx     |     |      │
│ ...                                │
├─────────────────────────┤
│    刹车线生产工序表       │
│ (第二个产品)             │
└─────────────────────────┘
```

### 4.3 版式B - 横向布局

- A4纸横向
- 每页4个产品
- 2行2列

```
┌───────────────┬───────────────┐
│   产品1        │   产品2        │
│   工序表       │   工序表       │
├───────────────┼───────────────┤
│   产品3        │   产品4        │
│   工序表       │   工序表       │
└───────────────┴───────────────┘
```

### 4.4 打印内容

每个产品打印卡片包含：

**表头信息：**
- 标题：刹车线生产工序表
- 产品名称 / 客户
- 产品编号 / 数量
- 产品规格 / 审核
- AI编号 / 批准

**工序表格：**
| 序号 | 工位 | 规格型号 | 外观 | 备注 |
|------|------|----------|------|------|

### 4.5 前端实现

使用CSS @media print 控制打印样式：

```css
@media print {
  @page {
    size: A4;
    margin: 10mm;
  }
  .print-card {
    page-break-inside: avoid;
  }
}
```

## 5. API变更

### 5.1 创建产品

`POST /api/products`

请求体新增字段映射：
```json
{
  "productName": "string",
  "productSpec": "string",
  "productCode": "string",
  "productDrawingNo": "string",
  "productAlias": "string",
  "productCategory": "string",
  "customer": "string",
  "quantity": "string",
  "review": "string",
  "approval": "string",
  "appearance": "string",
  "remark": "string",
  "status": "active",
  "processes": [
    { "processName": "xxx", "spec": "xxx", "unitPrice": 0, "sortOrder": 1 }
  ]
}
```

响应：返回完整产品信息（含ai_id）

### 5.2 获取产品详情

`GET /api/products/:id`

响应需包含所有字段：
```json
{
  "id": 1,
  "product_name": "xxx",
  "product_spec": "xxx",
  "ai_id": "AI5123456",
  "product_code": "xxx",
  "product_drawing_no": "xxx",
  "product_alias": "xxx",
  "product_category": "xxx",
  "customer": "xxx",
  "quantity": "xxx",
  "review": "xxx",
  "approval": "xxx",
  "appearance": "xxx",
  "remark": "xxx",
  "status": "active",
  "processes": []
}
```

## 6. 实现清单

| 序号 | 任务 | 位置 |
|------|------|------|
| 1 | 表单双列布局 | ProductList.vue |
| 2 | 字段完整带入 | loadProductDetail() |
| 3 | AI编码只读显示 | 表单 |
| 4 | 新增时支持工序编辑 | 对话框 |
| 5 | 复制按钮 | 操作栏 |
| 6 | 打印按钮 | 搜索栏 |
| 7 | 打印预览对话框 | 新增组件 |
| 8 | 打印样式 | CSS |
| 9 | 后端ai_id自动生成 | productController |

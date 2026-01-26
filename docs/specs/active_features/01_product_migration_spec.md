# 新东产品数据迁移规格说明

## 1. 概述

将 `doc/jsons/` 下的JSON文件导入到指定company_id的产品表中。

## 2. JSON数据结构

```json
{
  "sheet": "手刹线-脚刹线",      // 忽略
  "position": "R1C1",           // 忽略
  "product_name": "手刹线",
  "product_id": "H90/EVH9",
  "product_spec": "465mm",
  "ai_id": "五星钻豹1",          // 唯一标识
  "customer": "五星钻豹",
  "quantity": "",               // 数量
  "review": "",                 // 审核人
  "approval": "",               // 批准人
  "appearance": "黑",           // 产品级外观
  "remark": "",                 // 产品级备注
  "processes": [
    { "seq": 1, "station": "切线管", "spec": "0.395米/2.5黑皮" }
  ]
}
```

## 3. 数据库变更

### 3.1 products表新增字段

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| ai_id | VARCHAR(50) | NULL | AI编码，与company_id组合唯一 |
| quantity | VARCHAR(50) | NULL | 数量 |
| review | VARCHAR(100) | NULL | 审核人 |
| approval | VARCHAR(100) | NULL | 批准人 |
| appearance | VARCHAR(200) | NULL | 产品外观 |
| remark | VARCHAR(500) | NULL | 产品备注 |

> product_spec、customer字段已存在

### 3.2 product_processes表

已有字段满足需求：
- process_name ← station
- spec ← spec
- sort_order ← seq

**重要变更**：移除`uk_product_process(product_id, process_name)`唯一约束，因同一产品下工序名称可重复。

### 3.3 约束变更

| 表 | 约束名 | 字段 | 操作 |
|------|--------|------|------|
| products | uk_company_code | company_id + product_code | 移除 |
| products | uk_company_ai_code | company_id + ai_id | 新增 |
| product_processes | uk_product_process | product_id + process_name | 移除 |

## 4. 唯一性校验规则

- **产品唯一性**：以 `ai_id + company_id` 为唯一键
- **工序无唯一性约束**：同一产品下工序名称可重复，以seq区分

## 5. 字段映射

### 5.1 products表映射

| JSON字段 | 数据库字段 | 说明 |
|----------|------------|------|
| ai_id | ai_id | 唯一标识 |
| product_name | product_name | |
| product_id | product_code | |
| product_spec | product_spec | |
| customer | customer | |
| quantity | quantity | 新增字段 |
| review | review | 新增字段 |
| approval | approval | 新增字段 |
| appearance | appearance | 新增字段 |
| remark | remark | 新增字段 |
| - | company_id | 运行时指定 |
| - | status | 默认'active' |

### 5.2 product_processes表映射

| JSON字段 | 数据库字段 | 说明 |
|----------|------------|------|
| processes[].seq | sort_order | |
| processes[].station | process_name | |
| processes[].spec | spec | 允许空 |
| - | unit_price | 默认0.00 |
| - | status | 默认'active' |

### 5.3 忽略字段

sheet, position

## 6. 导入逻辑

### 6.1 冲突处理策略

```
对每条JSON产品记录：
1. 查询 ai_id + company_id 是否存在
2. 存在：
   a. 更新products表（除id和company_id外的字段）
   b. 删除该产品下所有工序
   c. 重新插入工序
3. 不存在：
   a. 插入products
   b. 插入product_processes
```

### 6.2 事务控制

- 单条产品为一个事务单元
- 失败记录日志，不影响其他产品

## 7. SQL迁移脚本

### 7.1 DDL变更

```sql
-- products表新增字段
ALTER TABLE products 
ADD COLUMN ai_id VARCHAR(50) NULL COMMENT 'AI编码' AFTER product_code,
ADD COLUMN quantity VARCHAR(50) NULL COMMENT '数量' AFTER customer,
ADD COLUMN review VARCHAR(100) NULL COMMENT '审核人' AFTER quantity,
ADD COLUMN approval VARCHAR(100) NULL COMMENT '批准人' AFTER review,
ADD COLUMN appearance VARCHAR(200) NULL COMMENT '产品外观' AFTER approval,
ADD COLUMN remark VARCHAR(500) NULL COMMENT '产品备注' AFTER appearance;

-- products表移除旧约束并添加新约束
ALTER TABLE products DROP INDEX IF EXISTS uk_company_code;
ALTER TABLE products ADD UNIQUE KEY uk_company_ai_code (company_id, ai_id);

-- product_processes表移除唯一约束
ALTER TABLE product_processes DROP INDEX IF EXISTS uk_product_process;
```

### 7.2 幂等性DDL（推荐）

```sql
-- products.ai_id
SET @sql = IF(
  (SELECT COUNT(*) FROM information_schema.COLUMNS 
   WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = 'products' AND COLUMN_NAME = 'ai_id') = 0,
  'ALTER TABLE products ADD COLUMN ai_id VARCHAR(50) NULL COMMENT ''AI编码'' AFTER product_code',
  'SELECT 1'
);
PREPARE stmt FROM @sql; EXECUTE stmt; DEALLOCATE PREPARE stmt;

-- products.quantity
SET @sql = IF(
  (SELECT COUNT(*) FROM information_schema.COLUMNS 
   WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = 'products' AND COLUMN_NAME = 'quantity') = 0,
  'ALTER TABLE products ADD COLUMN quantity VARCHAR(50) NULL COMMENT ''数量'' AFTER customer',
  'SELECT 1'
);
PREPARE stmt FROM @sql; EXECUTE stmt; DEALLOCATE PREPARE stmt;

-- products.review
SET @sql = IF(
  (SELECT COUNT(*) FROM information_schema.COLUMNS 
   WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = 'products' AND COLUMN_NAME = 'review') = 0,
  'ALTER TABLE products ADD COLUMN review VARCHAR(100) NULL COMMENT ''审核人'' AFTER quantity',
  'SELECT 1'
);
PREPARE stmt FROM @sql; EXECUTE stmt; DEALLOCATE PREPARE stmt;

-- products.approval
SET @sql = IF(
  (SELECT COUNT(*) FROM information_schema.COLUMNS 
   WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = 'products' AND COLUMN_NAME = 'approval') = 0,
  'ALTER TABLE products ADD COLUMN approval VARCHAR(100) NULL COMMENT ''批准人'' AFTER review',
  'SELECT 1'
);
PREPARE stmt FROM @sql; EXECUTE stmt; DEALLOCATE PREPARE stmt;

-- products.appearance
SET @sql = IF(
  (SELECT COUNT(*) FROM information_schema.COLUMNS 
   WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = 'products' AND COLUMN_NAME = 'appearance') = 0,
  'ALTER TABLE products ADD COLUMN appearance VARCHAR(200) NULL COMMENT ''产品外观'' AFTER approval',
  'SELECT 1'
);
PREPARE stmt FROM @sql; EXECUTE stmt; DEALLOCATE PREPARE stmt;

-- products.remark
SET @sql = IF(
  (SELECT COUNT(*) FROM information_schema.COLUMNS 
   WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = 'products' AND COLUMN_NAME = 'remark') = 0,
  'ALTER TABLE products ADD COLUMN remark VARCHAR(500) NULL COMMENT ''产品备注'' AFTER appearance',
  'SELECT 1'
);
PREPARE stmt FROM @sql; EXECUTE stmt; DEALLOCATE PREPARE stmt;

-- uk_company_code移除
SET @sql = IF(
  (SELECT COUNT(*) FROM information_schema.STATISTICS 
   WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = 'products' AND INDEX_NAME = 'uk_company_code') > 0,
  'ALTER TABLE products DROP INDEX uk_company_code',
  'SELECT 1'
);
PREPARE stmt FROM @sql; EXECUTE stmt; DEALLOCATE PREPARE stmt;

-- uk_company_ai_code
SET @sql = IF(
  (SELECT COUNT(*) FROM information_schema.STATISTICS 
   WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = 'products' AND INDEX_NAME = 'uk_company_ai_code') = 0,
  'ALTER TABLE products ADD UNIQUE KEY uk_company_ai_code (company_id, ai_id)',
  'SELECT 1'
);
PREPARE stmt FROM @sql; EXECUTE stmt; DEALLOCATE PREPARE stmt;

-- 移除product_processes的uk_product_process
SET @sql = IF(
  (SELECT COUNT(*) FROM information_schema.STATISTICS 
   WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = 'product_processes' AND INDEX_NAME = 'uk_product_process') > 0,
  'ALTER TABLE product_processes DROP INDEX uk_product_process',
  'SELECT 1'
);
PREPARE stmt FROM @sql; EXECUTE stmt; DEALLOCATE PREPARE stmt;
```

## 8. 验证检查

```sql
-- ai_id唯一性检查
SELECT ai_id, company_id, COUNT(*) as cnt 
FROM products WHERE company_id = ? 
GROUP BY ai_id, company_id HAVING cnt > 1;

-- 产品总数
SELECT COUNT(*) FROM products WHERE company_id = ?;

-- 工序总数
SELECT COUNT(*) FROM product_processes pp 
JOIN products p ON pp.product_id = p.id WHERE p.company_id = ?;
```

## 9. 回滚方案

```sql
-- 删除数据
DELETE FROM product_processes WHERE product_id IN (SELECT id FROM products WHERE company_id = ?);
DELETE FROM products WHERE company_id = ?;

-- 回滚DDL
ALTER TABLE products DROP COLUMN ai_id, DROP COLUMN quantity, DROP COLUMN review, DROP COLUMN approval, DROP COLUMN appearance, DROP COLUMN remark;
ALTER TABLE products DROP INDEX uk_company_ai_code;
ALTER TABLE product_processes ADD UNIQUE KEY uk_product_process (product_id, process_name);
```

## 10. 目标公司

- company_id: 5
- company_name: 智涌纪元科技公司

### 10.1 导入前清空数据

```sql
DELETE FROM product_processes WHERE product_id IN (SELECT id FROM products WHERE company_id = 5);
DELETE FROM products WHERE company_id = 5;
```

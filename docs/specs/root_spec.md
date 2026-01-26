# 孺子牛小工单 - 根规范 (Root Specification)

> 本文档是项目的"宪法"，定义全局共识。所有端必须遵守。
> **任何功能变更必须先更新本规范，再进行开发。**

---

## 1. 业务目标

孺子牛小工单是面向工厂的轻量级工单管理系统，核心目标：

1. **工单数字化**：维修、不合格品、计件薪资、日报等工单的录入、审批、统计
2. **权限精细化**：基于 RBAC 的多角色权限控制
3. **多租户隔离**：企业间数据完全隔离

---

## 2. 技术栈

| 端 | 技术栈 | 仓库路径 |
|---|-------|---------|
| Backend | Node.js + Express + MySQL | `industry_plat_server/backend/` |
| Web Admin | Vue3 + Vite + Element Plus + Pinia | `industry_plat_server/frontend/` |
| App | Flutter + Riverpod + GoRouter | `industry_plat_flutter/` |

---

## 3. 多租户规则

### 3.1 数据隔离

- 所有业务表必须包含 `company_id` 字段
- 查询时必须携带 `company_id` 条件（除系统管理员）
- 禁止跨企业数据访问

### 3.2 特殊企业

| company_id | 名称 | 说明 |
|------------|------|------|
| 1 | 系统 | 存放系统级角色和系统管理员 |

---

## 4. 认证与授权

### 4.1 认证流程

```
[登录] POST /api/auth/login
    ↓
[返回 Token] { token: "xxx", user: {...}, permissions: [...] }
    ↓
[请求携带] Authorization: Bearer <token>
    ↓
[Token 永久有效] 除非用户主动退出或账号被禁用
```

### 4.2 Token 规范

| 项目 | 值 |
|-----|---|
| 算法 | HS256 |
| 有效期 | 永久（用户不主动退出一直有效） |
| Payload | `{ id: userId }` |
| 刷新机制 | 无 |

### 4.3 RBAC 权限模型

#### 角色层级

```
系统管理员 (system_admin)
    ↓ 管理所有企业
企业管理员 (company_admin)
    ↓ 管理本企业所有事务
自定义角色
    ↓ 按权限配置访问
```

#### 权限模块

| 模块 | module 值 | 说明 |
|-----|----------|------|
| 维修 | repair | 维修工单 |
| 不合格品 | nonconforming | 不合格品记录 |
| 模具 | mold | 模具管理 |
| 计件 | piecework | 计件薪资 |
| 日报 | daily_report | 日报管理 |
| 用户 | user | 用户管理 |
| 角色 | role | 角色管理 |

#### 权限动作

| 动作 | 字段 | 说明 |
|-----|------|------|
| 查看本人 | can_view_own | 仅查看自己创建的数据 |
| 查看部门 | can_view_dept | 查看本企业所有数据 |
| 创建 | can_create | 新增记录 |
| 修改 | can_update | 编辑记录 |
| 删除 | can_delete | 删除记录 |
| 审核 | can_audit | 审批记录 |
| 导出 | can_export | 导出数据 |
| 统计 | can_statistics | 查看统计图表 |

---

## 5. 通用标准

### 5.1 API 响应格式

```json
{
  "code": 200,
  "message": "操作成功",
  "data": {}
}
```

### 5.2 HTTP 状态码

| 状态码 | 含义 | 使用场景 |
|-------|------|---------|
| 200 | 成功 | GET/PUT/PATCH/DELETE 成功 |
| 201 | 创建成功 | POST 创建资源成功 |
| 400 | 参数错误 | 请求参数校验失败 |
| 401 | 未认证 | Token 无效或过期 |
| 403 | 无权限 | 权限不足 |
| 404 | 不存在 | 资源未找到 |
| 500 | 服务器错误 | 服务端异常 |

### 5.3 分页协议

**请求参数：**
| 参数 | 类型 | 默认值 | 说明 |
|-----|------|-------|------|
| page | number | 1 | 页码（从1开始） |
| pageSize | number | 20 | 每页条数 |

**响应格式：**
```json
{
  "code": 200,
  "data": {
    "list": [],
    "total": 100,
    "page": 1,
    "pageSize": 20
  }
}
```

### 5.4 时间格式

| 场景 | 格式 | 示例 |
|-----|------|------|
| 日期字段 | YYYY-MM-DD | 2025-01-13 |
| 时间字段 | YYYY-MM-DD HH:mm:ss | 2025-01-13 14:30:00 |
| 时间戳存储 | MySQL TIMESTAMP | - |

### 5.5 审核状态

| 状态 | 值 | 说明 |
|-----|---|------|
| 待审核 | pending | 默认状态 |
| 已通过 | approved | 审核通过 |
| 已驳回 | rejected | 审核驳回 |

---

## 6. 数据库规范

### 6.1 命名规范

| 类型 | 规范 | 示例 |
|-----|------|------|
| 表名 | 小写复数 | users, repairs |
| 字段名 | 小写下划线，≤20字符 | company_id, created_at |
| 索引名 | idx_表名_字段 | idx_repairs_company |
| 外键名 | fk_表名_关联表 | fk_users_companies |

### 6.2 公共字段

所有业务表必须包含：

```sql
company_id INT NOT NULL COMMENT '企业ID',
creator_id INT NOT NULL COMMENT '创建人ID',
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
INDEX idx_company (company_id),
FOREIGN KEY (company_id) REFERENCES companies(id) ON DELETE CASCADE
```

### 6.3 迁移脚本规范 (Knex.js)

禁止使用 SQL 别名欺骗：严禁在 service 层通过 '--' as field 或在代码层手动对 Entity 赋值来“伪造”新字段。

物理存储优先：任何在 API Spec 或 UI 中新增的业务字段，必须首先评估其存储归属。若该字段属于某个领域模型（如 Product），必须通过 Knex Migration 在物理表中创建该字段。

**禁止回滚原则**：永远不要使用 `npm run migrate:rollback` 进行数据库变更。所有字段删除、表结构调整必须通过新建正向迁移脚本实现，确保变更历史可追溯且生产环境安全。

- 路径：`backend/migrations/`
- 命名：`YYYYMMDDHHMMSS_description.js`
- 示例：`20250115120000_add_user_avatar.js`

**迁移命令：**
```bash
npm run migrate          # 执行所有待迁移
npm run migrate:status   # 查看迁移状态
```

**脚本模板：**
```javascript
exports.up = async function(knex) {
  const hasTable = await knex.schema.hasTable('table_name');
  if (!hasTable) {
    return knex.schema.createTable('table_name', (table) => {
      table.increments('id').primary();
      // ...
    });
  }
};

exports.down = function(knex) {
  return knex.schema.dropTableIfExists('table_name');
};
```

**变更登记**：所有 DDL 变更必须在 `doc/specs/database_ledger.md` 登记。

---

## 7. Git 分支策略

```
develop (开发) → test (测试) → master (生产)
```

| 分支 | 用途 | 直接 push |
|-----|------|----------|
| master | 生产发布 | 禁止 |
| test | 测试验证 | 禁止 |
| develop | 日常开发 | 允许 |

### 提交规范

格式：`<type>(<scope>): <subject>`

| type | 说明 |
|------|------|
| feat | 新功能 |
| fix | 修复 bug |
| docs | 文档更新 |
| refactor | 重构 |
| perf | 性能优化 |
| test | 测试 |
| chore | 构建/工具 |

---

## 8. 环境配置

| 环境 | 前端端口 | 后端端口 | 数据库 |
|-----|---------|---------|-------|
| 开发 | 8080 | 3001 | factory_db_dev |
| 测试 | 8082 | 3003 | factory_db_test |
| 生产 | 8081 | 3002 | factory_db_prod |

---

## 9. 规范维护

1. 功能变更前，先更新相关 spec 文档
2. 代码评审时，检查 spec 是否同步更新
3. spec 变更需在 commit message 中注明

---

**相关文档：**
- [API 规范](./api_spec.md)
- [后端规范](./backend_spec.md)
- [管理端规范](./web_admin_spec.md)
- [App 规范](./app_spec.md)

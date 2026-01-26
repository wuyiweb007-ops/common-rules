# 孺子牛小工单 - 后端规范 (Backend Specification)

> 本文档定义后端开发规范，包括目录结构、分层职责、数据库模型等。
> **后端改动必须先更新本文档，再进行开发。**

---

## 1. 目录结构

```
backend/
├── src/
│   ├── app.js              # 应用入口
│   ├── config/
│   │   ├── database.js     # 数据库配置
│   │   ├── jwt.js          # JWT 配置
│   │   ├── init.sql        # 初始化脚本
│   │   └── migrations/     # 迁移脚本
│   ├── controllers/        # 控制器层
│   ├── services/           # 服务层
│   ├── middleware/         # 中间件
│   ├── routes/             # 路由定义
│   ├── utils/              # 工具函数
│   └── scripts/            # 运维脚本
├── downloads/              # APK/wgt 下载目录
├── public/                 # 静态资源
└── package.json
```

---

## 2. 分层职责

### 2.1 Controller（控制器）

**职责：**
- 接收请求，提取参数
- 调用 Service 处理业务
- 返回响应

**规范：**
- 不包含业务逻辑
- 使用 `utils/response.js` 统一响应格式
- 文件命名：`xxxController.js`

```javascript
// 示例
async function getRepairList(req, res) {
  const { page, pageSize, workshop } = req.query;
  const result = await repairService.getList(req.user, { page, pageSize, workshop });
  success(res, result);
}
```

### 2.2 Service（服务）

**职责：**
- 核心业务逻辑
- 数据库操作
- 权限数据过滤

**规范：**
- 接收 `user` 对象，处理数据隔离
- 返回纯数据，不操作 response
- 文件命名：`xxxService.js`

```javascript
// 示例
async function getList(user, { page, pageSize, workshop }) {
  const { companyId } = user;
  let sql = 'SELECT * FROM repairs WHERE company_id = ?';
  // ... 业务逻辑
}
```

### 2.3 Middleware（中间件）

| 中间件 | 文件 | 职责 |
|-------|------|------|
| auth | auth.js | Token 验证，附加 req.user |
| permission | permission.js | 权限检查 |
| dataFilter | dataFilter.js | 数据范围过滤 |
| errorHandler | errorHandler.js | 全局错误处理 |
| logger | logger.js | 请求日志 |
| upload | upload.js | 文件上传 |

### 2.4 Routes（路由）

**规范：**
- RESTful 风格
- 包含 Swagger 注释
- 文件命名：`xxxs.js`（复数）

---

## 3. 数据库模型

### 3.1 核心表结构

#### companies（企业表）
| 字段 | 类型 | 说明 |
|-----|------|------|
| id | INT PK | 企业ID |
| name | VARCHAR(100) | 企业名称 |
| status | ENUM | active/inactive |
| auto_approve | TINYINT(1) | 自动审批 |
| created_at | TIMESTAMP | 创建时间 |

#### users（用户表）
| 字段 | 类型 | 说明 |
|-----|------|------|
| id | INT PK | 用户ID |
| company_id | INT FK | 企业ID |
| username | VARCHAR(50) | 用户名（唯一） |
| password | VARCHAR(255) | 密码（bcrypt） |
| real_name | VARCHAR(50) | 真实姓名 |
| role_id | INT FK | 角色ID |
| dept | VARCHAR(50) | 部门 |
| status | ENUM | active/inactive |

#### roles（角色表）
| 字段 | 类型 | 说明 |
|-----|------|------|
| id | INT PK | 角色ID |
| company_id | INT FK | 企业ID |
| role_name | VARCHAR(50) | 角色名称 |
| role_code | VARCHAR(50) | 角色代码 |
| is_system | TINYINT(1) | 系统角色标记 |
| status | ENUM | active/inactive |

#### role_perms（角色权限表）
| 字段 | 类型 | 说明 |
|-----|------|------|
| id | INT PK | 权限ID |
| role_id | INT FK | 角色ID |
| module | VARCHAR(50) | 模块名 |
| can_view_own | TINYINT(1) | 查看本人 |
| can_view_dept | TINYINT(1) | 查看部门 |
| can_create | TINYINT(1) | 创建 |
| can_update | TINYINT(1) | 修改 |
| can_delete | TINYINT(1) | 删除 |
| can_audit | TINYINT(1) | 审核 |
| can_export | TINYINT(1) | 导出 |
| can_statistics | TINYINT(1) | 统计 |

### 3.2 业务表结构

#### repairs（维修表）
| 字段 | 类型 | 说明 |
|-----|------|------|
| id | INT PK | 维修ID |
| company_id | INT FK | 企业ID |
| repair_date | DATE | 维修日期 |
| workshop | VARCHAR(50) | 车间 |
| equip_name | VARCHAR(100) | 设备名称 |
| equip_code | VARCHAR(50) | 设备编号 |
| fault_desc | TEXT | 故障现象 |
| fault_reason | TEXT | 故障原因 |
| repair_content | TEXT | 维修内容 |
| repair_result | TEXT | 维修结果 |
| repairer | VARCHAR(50) | 维修人 |
| submitter | VARCHAR(50) | 送修人 |
| status | ENUM | pending/approved/rejected |
| creator_id | INT FK | 创建人ID |
| auditor_id | INT | 审核人ID |

#### pieceworks（计件薪资表）
| 字段 | 类型 | 说明 |
|-----|------|------|
| id | INT PK | 计件ID |
| company_id | INT FK | 企业ID |
| record_code | VARCHAR(50) | 记录编号 |
| emp_name | VARCHAR(50) | 员工姓名 |
| dept | VARCHAR(50) | 部门 |
| product_id | INT FK | 产品ID |
| product_name | VARCHAR(100) | 产品名称（冗余） |
| product_code | VARCHAR(50) | 产品编号（冗余） |
| process_name | VARCHAR(50) | 工序名称 |
| work_date | DATE | 生产日期 |
| quantity | INT | 计件数量 |
| unit | VARCHAR(20) | 单位 |
| unit_price | DECIMAL(10,2) | 单价 |
| salary_amt | DECIMAL(10,2) | 薪资金额 |
| creator_id | INT FK | 创建人ID |
| finance_status | ENUM | pending/verified/rejected |

#### products（产品表）
| 字段 | 类型 | 说明 |
|-----|------|------|
| id | INT PK | 产品ID |
| company_id | INT FK | 企业ID |
| product_name | VARCHAR(100) | 产品名称 |
| product_code | VARCHAR(50) | 产品编号(可选) |
| ai_id | VARCHAR(50) | AI编码(唯一标识) |
| product_drawing_no | VARCHAR(50) | 产品图号(可选) |
| product_spec | VARCHAR(200) | 规格 |
| customer | VARCHAR(100) | 客户 |
| quantity | VARCHAR(50) | 数量 |
| review | VARCHAR(100) | 审核人 |
| approval | VARCHAR(100) | 批准人 |
| appearance | VARCHAR(200) | 产品外观 |
| remark | VARCHAR(500) | 产品备注 |
| status | ENUM | active/inactive |

**约束**：
- `uk_company_ai_code(company_id, ai_id)` - 企业内AI编码唯一
- `uk_company_name(company_id, product_name)` - 已移除

#### product_processes（产品工序表）
| 字段 | 类型 | 说明 |
|-----|------|------|
| id | INT PK | 工序ID |
| product_id | INT FK | 产品ID |
| process_name | VARCHAR(50) | 工序名称 |
| spec | VARCHAR(500) | 工序规格说明 |
| appearance | VARCHAR(200) | 外观要求 |
| remark | VARCHAR(500) | 备注 |
| unit_price | DECIMAL(10,2) | 单价(默认0.00) |
| sort_order | INT | 排序 |
| status | ENUM | active/inactive |

**约束**：
- `uk_product_process(product_id, process_name)` - 已移除(允许同名工序)

#### dictionaries（字典表）
| 字段 | 类型 | 说明 |
|-----|------|------|
| id | INT PK | 字典ID |
| company_id | INT FK | 企业ID |
| category | VARCHAR(50) | 类别 |
| value | VARCHAR(100) | 值 |
| label | VARCHAR(100) | 标签 |
| sort_order | INT | 排序 |
| is_active | TINYINT(1) | 是否启用 |

### 3.3 表关系图

```
companies (1) ──< users (N)
    │                │
    │                └──> roles (N:1)
    │                         │
    │                         └──< role_perms (N)
    │
    ├──< repairs (N)
    ├──< nonconforming (N)
    ├──< pieceworks (N) ──> products (N:1)
    ├──< molds (N)
    ├──< products (N) ──< product_processes (N)
    └──< dictionaries (N)
```

---

## 4. 字符集配置

### 4.1 数据库级别

```sql
CREATE DATABASE factory_db_xxx CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### 4.2 连接池配置

`database.js` 中配置：
```javascript
charset: 'utf8mb4'
```

### 4.3 迁移脚本字符集

执行迁移前必须设置：
```sql
SET NAMES utf8mb4;
```

### 4.4 数据导入注意事项

- JSON 源数据可能包含换行符(`\n`)，需正确处理
- 中文字符确保使用 UTF-8 编码
- 导入时每个连接需执行 `SET NAMES utf8mb4`

---

## 5. 迁移脚本规范

### 5.1 命名规范

```
NNN_description.sql

示例：
001_create_app_versions_table.sql
002_add_download_page_flag.sql
010_add_soft_delete.sql
```

### 5.2 脚本结构

```sql
-- 描述：添加 xxx 功能
-- 作者：xxx
-- 日期：2025-01-13

-- 1. 新增表/字段
ALTER TABLE xxx ADD COLUMN yyy VARCHAR(50);

-- 2. 数据迁移（如需要）
UPDATE xxx SET yyy = 'default' WHERE yyy IS NULL;

-- 3. 添加索引（如需要）
CREATE INDEX idx_xxx_yyy ON xxx(yyy);
```

### 5.3 执行方式

```bash
node src/scripts/runMigrations.js
```

---

## 6. 权限检查规范

### 6.1 中间件使用

```javascript
// 企业管理员检查
router.use(isCompanyAdmin);

// 系统管理员检查
router.use(isSystemAdmin);

// 模块权限检查
router.get('/', checkModulePermission('repair', 'view_own'), controller.getList);

// 创建权限
router.post('/', canCreate('piecework'), controller.create);

// 审核权限
router.patch('/:id/audit', canAudit('repair'), controller.audit);
```

### 6.2 数据过滤

```javascript
// Service 中根据权限过滤数据
async function getList(user, params) {
  const { companyId, id: userId } = user;
  
  // 查询用户权限
  const perms = await getPermissions(user.roleId, 'repair');
  
  let sql = 'SELECT * FROM repairs WHERE company_id = ?';
  
  // can_view_dept: 查看全部
  // can_view_own: 仅查看自己创建的
  if (!perms.can_view_dept && perms.can_view_own) {
    sql += ' AND creator_id = ?';
  }
}
```

---

## 7. 响应格式规范

### 7.1 使用方式

```javascript
const { success, created, badRequest, unauthorized, forbidden, notFound } = require('../utils/response');

// 成功
success(res, data);
success(res, data, '操作成功');

// 创建成功
created(res, { id: 1 });

// 参数错误
badRequest(res, '参数错误：xxx 不能为空');

// 未认证
unauthorized(res, 'Token 已过期');

// 无权限
forbidden(res, '无权限访问该资源');

// 不存在
notFound(res, '记录不存在');
```

---

## 8. 日志规范

### 8.1 日志级别

| 级别 | 使用场景 |
|-----|---------|
| error | 异常、错误 |
| warn | 警告信息 |
| info | 关键业务信息 |
| debug | 调试信息 |

### 8.2 使用方式

```javascript
const logger = require('../utils/logger');

logger.info('用户登录成功', { userId: 1, username: 'admin' });
logger.error('数据库查询失败', error);
```

---

## 9. 定时任务

### 9.1 调度器

文件：`services/schedulerService.js`

```javascript
// 启动所有调度器
schedulerService.startAllSchedulers();

// 停止所有调度器
schedulerService.stopAllSchedulers();
```

---

## 10. 环境配置

### 10.1 环境变量

| 变量 | 说明 | 默认值 |
|-----|------|-------|
| PORT | 服务端口 | 3000 |
| DB_HOST | 数据库地址 | localhost |
| DB_PORT | 数据库端口 | 3306 |
| DB_NAME | 数据库名 | factory_db_dev |
| DB_USER | 数据库用户 | root |
| DB_PASSWORD | 数据库密码 | - |
| JWT_SECRET | JWT 密钥 | - |
| JWT_EXPIRES_IN | Token 有效期 | 24h |

---

**相关文档：**
- [根规范](./root_spec.md)
- [API 规范](./api_spec.md)

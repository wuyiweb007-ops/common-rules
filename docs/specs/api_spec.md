# 孺子牛小工单 - API 规范 (API Specification)

> 本文档定义所有端必须遵守的 API 契约。
> **接口变更必须先更新本文档，再进行开发。**

---

## 1. 通用协议

### 1.1 请求规范

**请求头：**
```
Authorization: Bearer <token>
Content-Type: application/json
```

**分页参数：**
| 参数 | 类型 | 默认值 | 说明 |
|-----|------|-------|------|
| page | number | 1 | 页码（从1开始） |
| pageSize | number | 20 | 每页条数 |

**日期筛选参数：**
| 参数 | 格式 | 说明 |
|-----|------|------|
| startDate | YYYY-MM-DD | 开始日期 |
| endDate | YYYY-MM-DD | 结束日期 |

### 1.2 响应格式

**成功响应：**
```json
{
  "code": 200,
  "message": "操作成功",
  "data": {}
}
```

**分页响应：**
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

**错误响应：**
```json
{
  "code": 401,
  "message": "未认证",
  "data": null
}
```

### 1.3 HTTP 状态码

| 状态码 | code 值 | 使用场景 |
|-------|--------|---------|
| 200 | 200 | GET/PUT/PATCH/DELETE 成功 |
| 201 | 201 | POST 创建成功 |
| 400 | 400 | 参数错误 |
| 401 | 401 | 未认证（Token 无效/过期） |
| 403 | 403 | 无权限 |
| 404 | 404 | 资源不存在 |
| 500 | 500 | 服务器错误 |

---

## 2. 认证接口

### 2.1 登录

```
POST /api/auth/login
```

**请求体：**
```json
{
  "username": "admin",
  "password": "admin123"
}
```

**响应：**
```json
{
  "code": 200,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": 1,
      "username": "admin",
      "realName": "管理员",
      "role": "company_admin",
      "companyId": 2,
      "companyName": "测试工厂A",
      "dept": "管理部"
    },
    "permissions": [
      { "module": "repair", "can_view_own": 1, "can_view_dept": 1, "can_create": 1, ... }
    ]
  }
}
```

### 2.2 获取当前用户信息

```
GET /api/auth/profile
```

**响应：** 同登录响应的 user + permissions 部分

### 2.3 登出

```
POST /api/auth/logout
```

---

## 3. 用户管理

**基础路径：** `/api/users`
**权限要求：** 企业管理员（company_admin）

| 方法 | 路径 | 说明 |
|-----|------|------|
| GET | / | 用户列表（支持 keyword、role 筛选） |
| GET | /:id | 用户详情 |
| POST | / | 创建用户 |
| PUT | /:id | 完整更新用户 |
| PATCH | /:id | 部分更新用户 |
| DELETE | /:id | 删除用户 |
| POST | /:id/restore | 恢复用户 |
| POST | /:id/reset-password | 重置密码为 123456 |
| PATCH | /:id/role | 分配角色 |
| POST | /change-password | 修改当前用户密码 |
| POST | /reset-own-password | 重置自己密码 |

**创建用户请求体：**
```json
{
  "username": "worker01",
  "password": "123456",
  "realName": "张三",
  "roleId": 3,
  "dept": "生产车间"
}
```

---

## 4. 角色管理

**基础路径：** `/api/roles`
**权限要求：** 企业管理员

| 方法 | 路径 | 说明 |
|-----|------|------|
| GET | / | 角色列表 |
| GET | /modules | 权限模块列表 |
| GET | /:id | 角色详情（含权限） |
| POST | / | 创建角色 |
| PUT | /:id | 更新角色基本信息 |
| PATCH | /:id/permissions | 更新角色权限 |
| PATCH | /:id/status | 修改角色状态 |
| POST | /:id/copy | 复制角色 |
| DELETE | /:id | 删除角色 |
| POST | /:id/restore | 恢复角色 |

**权限配置格式：**
```json
{
  "permissions": [
    {
      "module": "repair",
      "can_view_own": 1,
      "can_view_dept": 1,
      "can_create": 1,
      "can_update": 1,
      "can_delete": 0,
      "can_audit": 0,
      "can_export": 1,
      "can_statistics": 1
    }
  ]
}
```

---

## 5. 维修管理

**基础路径：** `/api/repairs`
**权限模块：** repair

| 方法 | 路径 | 说明 | 权限 |
|-----|------|------|------|
| GET | / | 维修列表 | can_view_own/can_view_dept |
| GET | /statistics | 统计数据 | can_statistics |
| GET | /export | 导出 Excel | can_export |
| GET | /:id | 详情 | can_view_own |
| POST | / | 创建 | can_create |
| PUT | /:id | 完整更新 | can_update |
| PATCH | /:id | 部分更新 | can_update |
| DELETE | /:id | 删除 | can_delete |
| POST | /:id/restore | 恢复 | can_delete |
| PATCH | /:id/audit | 审核 | can_audit |

**筛选参数：** workshop, status, startDate, endDate, keyword

---

## 6. 不合格品管理

**基础路径：** `/api/nonconforming`
**权限模块：** nonconforming

| 方法 | 路径 | 说明 | 权限 |
|-----|------|------|------|
| GET | / | 列表 | can_view_own/can_view_dept |
| GET | /statistics | 统计 | can_statistics |
| GET | /export | 导出 | can_export |
| GET | /:id | 详情 | can_view_own |
| POST | / | 创建 | can_create |
| PUT | /:id | 完整更新 | can_update |
| PATCH | /:id | 部分更新 | can_update |
| DELETE | /:id | 删除 | can_delete |
| POST | /:id/restore | 恢复 | can_delete |
| PATCH | /:id/audit | 审核 | can_audit |

**筛选参数：** deptShift, status, startDate, endDate

---

## 7. 计件薪资管理

**基础路径：** `/api/pieceworks`
**权限模块：** piecework

| 方法 | 路径 | 说明 | 权限 |
|-----|------|------|------|
| GET | / | 列表 | can_view_own/can_view_dept |
| GET | /statistics | 统计 | can_statistics |
| GET | /export | 导出 | can_export |
| GET | /:id | 详情 | can_view_own |
| POST | / | 创建 | can_create |
| PUT | /:id | 更新 | can_update |
| DELETE | /:id | 删除 | can_delete |
| POST | /:id/restore | 恢复 | can_delete |
| PATCH | /:id/reject | 核对驳回 | can_audit |
| PATCH | /:id/approve | 核对通过 | can_audit |
| POST | /batch-verify | 批量核对 | can_audit |
| POST | /batch-delete | 批量删除 | can_delete |

**筛选参数：** dept, workDateStart, workDateEnd, financeStatus, keyword

**创建请求体：**
```json
{
  "empName": "张三",
  "dept": "生产车间",
  "productId": 1,
  "productName": "产品A",
  "productCode": "P001",
  "processName": "工序1",
  "workDate": "2025-01-13",
  "quantity": 100,
  "unit": "件",
  "unitPrice": 0.50,
  "salaryAmt": 50.00
}
```

---

## 8. 日报管理

**基础路径：** `/api/daily-reports`
**权限模块：** daily_report

| 方法 | 路径 | 说明 | 权限 |
|-----|------|------|------|
| GET | / | 列表 | can_view_own/can_view_dept |
| GET | /by-date | 按日期聚合 | can_view_own |
| GET | /check-today | 检查今日是否已提交 | - |
| GET | /statistics | 统计 | can_statistics |
| GET | /export | 导出 | can_export |
| GET | /:id | 详情 | can_view_own |
| POST | / | 创建/追加 | can_create |
| PUT | /:id | 更新 | can_update |
| DELETE | /:id | 删除 | can_delete |
| POST | /:id/restore | 恢复 | can_delete |

**日报总结接口：**
| 方法 | 路径 | 说明 |
|-----|------|------|
| GET | /summaries | 总结列表 |
| POST | /summaries/generate | 生成单人总结 |
| POST | /summaries/generate-batch | 批量生成总结 |
| GET | /summaries/statistics | 提交统计 |
| DELETE | /summaries/:id | 删除总结 |

---

## 9. 计时工资管理

**基础路径：** `/api/time-wages`
**权限模块：** piecework（复用）

| 方法 | 路径 | 说明 | 权限 |
|-----|------|------|------|
| GET | / | 列表 | can_view_own/can_view_dept |
| GET | /statistics | 统计 | can_statistics |
| GET | /:id | 详情 | can_view_own |
| PUT | /:id | 更新（时间修正） | can_update |
| DELETE | /:id | 删除 | can_delete |
| POST | /:id/restore | 恢复 | can_delete |
| PATCH | /:id/verify | 财务核对 | can_audit |
| POST | /batch-verify | 批量核对 | can_audit |
| POST | /batch-delete | 批量删除 | can_delete |
| POST | /refresh | 从日报重新生成 | can_update |

**筛选参数：** workDateStart, workDateEnd, keyword, empName, dept

> **注意**：计时工资由后端从日报记录自动生成，无创建接口。

---

## 10. 产品管理

**基础路径：** `/api/products`
**权限模块：** product

| 方法 | 路径 | 说明 |
|-----|------|------|
| GET | / | 产品列表 |
| GET | /search | 搜索产品（下拉选择） |
| GET | /price | 获取工序单价 |
| GET | /by-code | 根据编号查询产品 |
| GET | /:id | 产品详情 |
| POST | / | 创建产品（支持含工序） |
| PUT | /:id | 更新产品 |
| PATCH | /:id/status | 更新状态 |
| DELETE | /:id | 删除产品 |
| POST | /:id/restore | 恢复产品 |

**创建产品请求体：**
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

> ai_id由后端自动生成，格式：`AI{company_id}{timestamp后6位}`

**产品详情响应：**
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

**产品工序接口：**
| 方法 | 路径 | 说明 |
|-----|------|------|
| GET | /:id/processes | 工序列表 |
| POST | /:id/processes | 创建工序 |
| PUT | /:id/processes/:processId | 更新工序 |
| DELETE | /:id/processes/:processId | 删除工序 |
| POST | /:id/processes/:processId/restore | 恢复工序 |

---

## 11. 模具管理

**基础路径：** `/api/molds`
**权限模块：** mold

| 方法 | 路径 | 说明 |
|-----|------|------|
| GET | / | 模具列表 |
| GET | /statistics | 统计 |
| GET | /export | 导出 |
| GET | /:id | 详情 |
| POST | / | 创建 |
| PUT | /:id | 更新 |
| DELETE | /:id | 删除 |
| POST | /:id/restore | 恢复 |

---

## 12. 字典管理

**基础路径：** `/api/dicts`
**权限要求：** 企业管理员

| 方法 | 路径 | 说明 |
|-----|------|------|
| GET | / | 字典列表 |
| GET | /categories | 字典类别列表 |
| GET | /options/:category | 按类别获取选项（下拉框） |
| GET | /:id | 详情 |
| POST | / | 创建 |
| PUT | /:id | 更新 |
| DELETE | /:id | 删除 |
| POST | /:id/restore | 恢复 |
| POST | /batch/sort | 批量排序 |
| POST | /batch/delete | 批量删除 |

**字典类别：**
| category | 说明 |
|----------|------|
| department | 部门 |
| workshop | 车间 |
| repair_result | 维修结果 |
| disposal_method | 处理方法 |
| unit | 单位 |

---

## 13. 企业管理

**基础路径：** `/api/companies`

| 方法 | 路径 | 说明 | 权限 |
|-----|------|------|------|
| GET | / | 企业列表 | system_admin |
| GET | /:id | 企业详情 | - |
| POST | / | 创建企业 | system_admin |
| PUT | /:id | 更新企业 | system_admin |
| PATCH | /:id/status | 修改状态 | system_admin |
| DELETE | /:id | 删除企业 | system_admin |
| POST | /:id/restore | 恢复企业 | system_admin |
| POST | /:id/reset-admin-password | 重置企业管理员密码 | system_admin |
| GET | /chat-welcome | 获取聊天开场白 | - |
| PUT | /chat-welcome | 更新聊天开场白 | company_admin |
| PATCH | /:id/auto-approve | 自动审批配置 | company_admin |

---

## 14. 系统配置

**基础路径：** `/api/configs`

| 方法 | 路径 | 说明 | 权限 |
|-----|------|------|------|
| GET | /public | 公开配置（Coze/火山等） | 无需认证 |
| GET | / | 全部配置 | 需认证 |
| POST | / | 设置配置 | system_admin |
| DELETE | /:key | 删除配置 | system_admin |

**公开配置示例**：
```json
{
  "coze_base_url": "https://api.coze.cn/v1/workflow/run",
  "coze_api_token": "xxx",
  "coze_piecework_workflow_id": "xxx",
  "volc_asr_app_id": "xxx",
  "volc_asr_access_token": "xxx"
}
```

---

## 15. APP 版本管理

**基础路径：** `/api/app/version`

| 方法 | 路径 | 说明 | 权限 |
|-----|------|------|------|
| GET | /check | 检查更新（App调用） | - |
| GET | / | 版本列表 | system_admin |
| POST | / | 发布版本 | system_admin |
| PUT | /:id | 更新版本 | system_admin |
| DELETE | /:id | 删除版本 | system_admin |

---

## 16. 健康检查

| 方法 | 路径 | 说明 |
|-----|------|------|
| GET | /health | 服务状态 |
| GET | /api/health | 服务+数据库状态 |

---

## 17. 审核状态枚举

| 状态 | 值 | 说明 |
|-----|---|------|
| 待审核 | pending | 默认状态 |
| 已通过 | approved | 审核通过 |
| 已驳回 | rejected | 审核驳回 |

**财务核对状态：**
| 状态 | 值 |
|-----|---|
| 待核对 | pending |
| 已核对 | verified |
| 已驳回 | rejected |

---

**相关文档：**
- [根规范](./root_spec.md)
- [后端规范](./backend_spec.md)

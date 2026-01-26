# 孺子牛小工单 - 管理端规范 (Web Admin Specification)

> 本文档定义 Vue3 管理后台开发规范。
> **前端改动必须先更新本文档，再进行开发。**

---

## 1. 目录结构

```
frontend/
├── src/
│   ├── api/                # API 接口
│   ├── components/         # 公共组件
│   ├── router/             # 路由配置
│   ├── store/              # Pinia 状态管理
│   ├── utils/              # 工具函数
│   ├── views/              # 页面组件
│   │   ├── admin/          # 企业管理页面
│   │   ├── dailyReport/    # 日报管理
│   │   ├── mold/           # 模具管理
│   │   ├── nonconforming/  # 不合格品管理
│   │   ├── piecework/      # 计件薪资
│   │   ├── product/        # 产品管理
│   │   ├── repair/         # 维修管理
│   │   ├── system/         # 系统管理
│   │   └── timeWage/       # 计时工资
│   ├── App.vue
│   └── main.js
├── index.html
└── vite.config.js
```

---

## 2. 技术栈

| 类型 | 技术 | 版本 |
|-----|------|------|
| 框架 | Vue3 | 3.x |
| 构建 | Vite | 5.x |
| UI | Element Plus | 2.x |
| 状态管理 | Pinia | 2.x |
| 路由 | Vue Router | 4.x |
| HTTP | Axios | 1.x |

---

## 3. 路由规范

### 3.1 路由结构

```javascript
{
  path: '/repair/list',
  name: 'RepairList',
  component: () => import('../views/repair/RepairList.vue'),
  meta: {
    title: '维修记录',
    requiresAuth: true,
    permission: { module: 'repair', action: 'can_view_dept' }
  }
}
```

### 3.2 路由权限配置

| meta 字段 | 类型 | 说明 |
|----------|------|------|
| requiresAuth | boolean | 是否需要登录 |
| roles | string[] | 角色限制（system_admin/company_admin） |
| permission | object | 模块权限 `{ module, action }` |
| permissionAny | object[] | 任一权限满足即可 |

### 3.3 路由表

#### 系统管理（system_admin）
| 路径 | 页面 |
|-----|------|
| /system/companies | 企业管理 |
| /system/app-versions | APP版本管理 |
| /system/configs | 配置管理 |
| /system/default-chat-welcome | 默认开场白 |
| /system/applications | 申请开通管理 |

#### 企业管理（company_admin）
| 路径 | 页面 |
|-----|------|
| /admin/users | 员工管理 |
| /admin/roles | 角色管理 |
| /admin/company-settings | 企业配置 |
| /admin/chat-welcome | 聊天开场白 |
| /custom-fields | 自定义字段 |
| /system/dict | 字典管理 |

#### 业务模块
| 路径 | 页面 | 权限模块 |
|-----|------|---------|
| /repair/list | 维修列表 | repair |
| /repair/create | 新增维修 | repair |
| /repair/edit/:id | 编辑维修 | repair |
| /repair/detail/:id | 维修详情 | repair |
| /repair/statistics | 维修统计 | repair |
| /nonconforming/list | 不合格品列表 | nonconforming |
| /piecework/list | 计件薪资列表 | piecework |
| /mold/list | 模具列表 | mold |
| /product/list | 产品列表 | product |
| /daily-report/list | 日报列表 | daily_report |

---

## 4. 状态管理

### 4.1 User Store

文件：`store/user.js`

**State：**
```javascript
{
  token: '',              // JWT Token
  userInfo: {},           // 用户信息
  permissions: {},        // 权限配置
  enabledModules: []      // 企业已开通模块
}
```

**Getters：**
| 名称 | 说明 |
|-----|------|
| isLoggedIn | 是否已登录 |
| role | 当前角色代码 |
| hasPermission(module, action) | 检查权限 |
| hasModule(moduleCode) | 检查模块是否开通 |

**Actions：**
| 名称 | 说明 |
|-----|------|
| login(data) | 登录 |
| logout() | 登出 |
| getUserInfo() | 获取用户信息 |

### 4.2 权限检查

```javascript
const userStore = useUserStore()

// 检查模块权限
if (userStore.hasPermission('repair', 'can_create')) {
  // 可以创建
}

// 检查审核权限
if (userStore.hasPermission('repair', 'can_audit')) {
  // 可以审核
}
```

---

## 5. API 规范

### 5.1 文件命名

```
api/
├── auth.js           # 认证接口
├── user.js           # 用户管理
├── role.js           # 角色管理
├── repair.js         # 维修管理
├── piecework.js      # 计件薪资
└── ...
```

### 5.2 接口定义

```javascript
import request from '@/utils/request'

// 获取列表
export function getRepairList(params) {
  return request({
    url: '/repairs',
    method: 'get',
    params
  })
}

// 创建
export function createRepair(data) {
  return request({
    url: '/repairs',
    method: 'post',
    data
  })
}

// 导出（Blob）
export function exportRepairs(params) {
  return request({
    url: '/repairs/export',
    method: 'get',
    params,
    responseType: 'blob'
  })
}
```

---

## 6. 页面规范

### 6.1 列表页结构

```vue
<template>
  <!-- 筛选区 -->
  <el-card class="filter-card">
    <el-form :inline="true">
      <!-- 筛选条件 -->
    </el-form>
  </el-card>
  
  <!-- 操作区 -->
  <div class="table-actions">
    <el-button type="primary" @click="handleCreate" v-if="canCreate">
      新增
    </el-button>
    <el-button @click="handleExport" v-if="canExport">
      导出
    </el-button>
  </div>
  
  <!-- 表格 -->
  <el-table :data="list" v-loading="loading">
    <!-- 列定义 -->
  </el-table>
  
  <!-- 分页 -->
  <el-pagination
    v-model:current-page="query.page"
    v-model:page-size="query.pageSize"
    :total="total"
    @current-change="fetchList"
  />
</template>
```

### 6.2 表单页结构

```vue
<template>
  <el-card>
    <el-form
      ref="formRef"
      :model="form"
      :rules="rules"
      label-width="100px"
    >
      <!-- 表单项 -->
      <el-form-item label="xxx" prop="xxx">
        <el-input v-model="form.xxx" />
      </el-form-item>
      
      <!-- 提交按钮 -->
      <el-form-item>
        <el-button type="primary" @click="handleSubmit">
          {{ isEdit ? '保存' : '创建' }}
        </el-button>
        <el-button @click="handleCancel">取消</el-button>
      </el-form-item>
    </el-form>
  </el-card>
</template>
```

---

## 7. 组件规范

### 7.1 公共组件

| 组件 | 说明 |
|-----|------|
| PermissionTree | 权限树选择器 |

### 7.2 组件命名

- 页面组件：`XxxList.vue`、`XxxForm.vue`、`XxxDetail.vue`
- 公共组件：`PascalCase.vue`

### 7.3 SearchFilter 搜索筛选组件

| Prop | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| showName | boolean | false | 显示姓名输入 |
| showDept | boolean | false | 显示部门筛选 |
| showPosition | boolean | false | 显示岗位筛选 |
| showStatus | boolean | false | 显示状态筛选 |
| statusOptions | array | [] | 状态选项 `[{label, value}]` |
| showDateRange | boolean | false | 显示日期范围 |

**日期快捷选项：**
| 分类 | 选项 |
|-----|------|
| 快捷 | 昨天、今天、近7天、近1月 |
| 周 | 本周、上周 |
| 月 | 本月、上月、近3月、近6月 |
| 季 | 本季、上季 |
| 年 | 本年、上年、近1年 |

**Events：**
| 事件 | 参数 | 说明 |
|-----|------|------|
| search | filters | 点击查询 |
| reset | - | 点击重置 |

### 7.4 DataTable 数据表格组件

| Prop | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| columns | array | [] | 列配置 |
| data | array | [] | 表格数据 |
| loading | boolean | false | 加载状态 |
| actions | object | {} | 操作栏配置 |
| selectable | boolean | false | 是否可多选 |
| pagination | object | null | 分页配置 `{page, pageSize, total}` |

**列配置 (column)：**
```js
{
  prop: 'field',
  label: '列名',
  width: 120,
  minWidth: 150,
  fixed: 'left'|'right',
  slot: 'slotName',
  formatter: (value, row) => string
}
```

**操作栏配置 (actions)：**
```js
{
  width: 200,
  view: true,
  edit: true|(row) => boolean,
  delete: true|(row) => boolean,
  approve: (row) => boolean,
  reject: (row) => boolean
}
```

**Events：**
| 事件 | 参数 | 说明 |
|-----|------|------|
| action | {type, row} | 操作按钮点击 |
| selection-change | rows | 选择变化 |
| page-change | {page, pageSize} | 分页变化 |

---

## 8. 工具函数

### 8.1 request.js

HTTP 请求封装：
- 自动携带 Token
- 401 自动跳转登录
- Blob 响应处理

### 8.2 formatter.js

格式化工具：
```javascript
import { formatDate, formatMoney } from '@/utils/formatter'

formatDate(date)        // 2025-01-13
formatMoney(100.5)      // ¥100.50
```

---

## 9. 表单验证

### 9.1 常用规则

```javascript
const rules = {
  // 必填
  name: [{ required: true, message: '请输入名称', trigger: 'blur' }],
  
  // 长度限制
  code: [
    { required: true, message: '请输入编号' },
    { max: 50, message: '最多50个字符' }
  ],
  
  // 数字
  quantity: [
    { required: true, message: '请输入数量' },
    { type: 'number', min: 1, message: '数量必须大于0' }
  ]
}
```

---

## 10. 审核状态展示

```vue
<el-tag :type="getStatusType(row.status)">
  {{ getStatusText(row.status) }}
</el-tag>

<script setup>
const statusMap = {
  pending: { text: '待审核', type: 'warning' },
  approved: { text: '已通过', type: 'success' },
  rejected: { text: '已驳回', type: 'danger' }
}

const getStatusText = (status) => statusMap[status]?.text || status
const getStatusType = (status) => statusMap[status]?.type || 'info'
</script>
```

---

## 11. 导出功能

```javascript
async function handleExport() {
  try {
    const res = await exportRepairs(query)
    const blob = new Blob([res.data], {
      type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'
    })
    const url = window.URL.createObjectURL(blob)
    const link = document.createElement('a')
    link.href = url
    link.download = `维修记录_${new Date().toISOString().slice(0,10)}.xlsx`
    link.click()
    window.URL.revokeObjectURL(url)
  } catch (error) {
    ElMessage.error('导出失败')
  }
}
```

---

## 12. 环境配置

### 12.1 环境变量

| 变量 | 说明 |
|-----|------|
| VITE_API_BASE_URL | API 基础路径 |
| VITE_BASE_PATH | 路由基础路径 |

### 12.2 配置文件

```javascript
// vite.config.js
export default defineConfig({
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:3001',
        changeOrigin: true
      }
    }
  }
})
```

---

## 13. 产品管理模块

### 13.1 产品表单

**双列布局：**
| 左列 | 右列 |
|------|------|
| 产品名称 * | 规格型号 * |
| AI编码 (只读) | 产品编号 * |
| 产品图号 | 商品别名 |
| 商品分类 | 客户名称 * |
| 数量 | 审核 |
| 批准 | 外观 |
| 备注 | 状态 |

**功能特性：**
- 新增/编辑共用同一对话框
- 新增时支持直接编辑工序
- 编辑时所有字段完整带入
- AI编码系统自动生成，只读显示

### 13.2 复制产品

- 操作栏增加"复制"按钮
- 复制当前产品信息（不含id、ai_id）
- 工序一并复制
- 保存时创建新产品

### 13.3 批量打印

**交互流程：**
1. 勾选产品
2. 点击打印按钮
3. 选择版式
4. 预览并打印

**版式A - 纵向：** A4纵向，每页2个产品
**版式B - 横向：** A4横向，每页4个产品（2x2）

**打印内容：**
- 表头：产品名称、客户、产品编号、产品规格、AI编号
- 工序表：序号、工位、规格型号、外观、备注

---

**相关文档：**
- [根规范](./root_spec.md)
- [API 规范](./api_spec.md)
- [产品表单与打印规范](./02_product_form_print_spec.md)

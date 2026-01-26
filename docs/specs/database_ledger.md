# 数据库变更总账 (Database Ledger)

> 本文档记录所有数据库 DDL 变更的设计意图和生命周期状态。
> **禁止在未登记本文档的情况下执行迁移脚本。**

---

## 生命周期状态说明

| 状态 | 含义 |
|------|------|
| Draft | 设计中，脚本未完成 |
| Ready | 脚本就绪，待执行 |
| Merged | 已合入主干，测试环境已验证 |
| Production | 已在生产环境执行 |

---

## 变更记录

| 版本 ID | 描述 | 状态 | 脚本文件 | 备注 |
|---------|------|------|----------|------|
| 20250115000000 | Baseline - 标记现有结构 | Ready | `migrations/20250115000000_baseline.js` | 首次引入 Knex，不执行实际 DDL |
| 20250115120000 | products 表新增 test 字段 | Ready | `migrations/20250115120000_add_test_to_products.js` | 默认值 "--" |
| 20250115130000 | products 表移除 test 字段 | Ready | `migrations/20250115130000_remove_test_from_products.js` | 正向删除字段，不使用回滚 |
| 20260118140000 | products 表新增总装工序字段，禁用现有产品工序 | Ready | `migrations/20260118140000_add_assembly_process_and_disable_processes.js` | 新增 assembly_process_name、assembly_unit_price 字段；将所有 product_processes 状态改为 inactive |
| 20260119093907 | pieceworks 表新增 product_spec 冗余字段 | Ready | `migrations/20260119093907_add_product_spec_to_pieceworks.js` | 在 product_code 后添加 product_spec 字段，方便查询 |

---

## 历史 SQL 脚本（已归档）

以下脚本在引入 Knex 之前手动执行，已包含在 baseline 中：

| 文件名 | 描述 |
|--------|------|
| 001_create_app_versions_table.sql | App 版本表 |
| 002_add_download_page_flag.sql | 下载页标记 |
| 003_add_trial_accounts.sql | 试用账户 |
| 004_add_trial_expire.sql | 试用过期 |
| 005_add_chat_welcome_messages.sql | 欢迎消息 |
| 006_add_applications.sql | 应用表 |
| 007_add_time_wages.sql | 计时工资 |
| 008_add_finance_fields_to_time_wages.sql | 工资财务字段 |
| 009_add_product_fields_and_base_processes.sql | 产品字段和基础工序 |
| 010_add_soft_delete.sql | 软删除 |
| 011_add_app_configs.sql | App 配置 |
| 012_add_default_chat_welcome.sql | 默认欢迎语 |
| 013_add_product_migration_fields.sql | 产品迁移字段 |
| 014_clear_company_100_data.sql | 清理公司100数据 |
| 015_verify_company_100_import.sql | 验证公司100导入 |
| 016_add_leave_hours_to_time_wages.sql | 请假时长字段 |
| add_company_modules.sql | 公司模块 |
| add_daily_report.sql | 日报 |
| add_dictionaries.sql | 字典表 |
| add_emp_name_to_daily_reports.sql | 日报员工姓名 |
| add_product_code_to_pieceworks.sql | 计件产品编码 |
| add_salary_type.sql | 薪资类型 |
| deploy_piecework_module_20251121.sql | 计件模块部署 |
| refactor_permissions.sql | 权限重构 |
| update_product_unique_constraints.sql | 产品唯一约束 |

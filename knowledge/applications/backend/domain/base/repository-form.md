---
type: domain-base
sub-type: repository-index
module: form
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/form/mapper/
owner: cordys-crm
verify-required: true
verify-note: 表单 JSON 结构/版本/字段权限映射 易变
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/base/api-form.md
---

# 自定义表单 存储索引（repository-form）

---

## 一、Mapper + 表定位

| ExtMapper 类名 | 职责一句话 | 表前缀（核对 @TableName） |
|---|---|---|
| **ExtFormDesignMapper**（或 ExtCustomFormMapper） | 表单定义 JSON（设计器）、版本、发布状态 | custom_form（或 crm_form_design，核对） |
| **ExtFormVersionMapper** | 表单版本快照（每次发布存一份） | custom_form_version |
| **ExtFormDataMapper** | 表单运行时数据（填好的表单实例），按 form_key + biz_id 关联业务 | custom_form_data |
| **ExtFormFieldPermissionMapper**（若独立） | 表单字段 角色可见/可编辑权限 配置（或走 sys_module_field_role_permission，核对） | custom_form_field_permission |
| **ExtFormTemplateMapper** | 表单模板（预置模板库） | custom_form_template |

XML 路径：`resources/mapper/form/Ext{Xxx}Mapper.xml`

> 注：若 Form 实际复用 system.ModuleField 而不是独立 Form 引擎，则以上表**可能只有 form_data 表，定义和字段都走 sys_module_field + sys_module**——核对代码实现后取哪套。

---

## 二、Flyway 脚本位置

`backend/crm/src/main/resources/db/migration/V{YYYYMMDDHHmm}__create_custom_form_tables.sql`

---

## 三、定位技巧

| 场景 | 找哪里 |
|---|---|
| 表单设计器保存的 JSON 存在哪 | ExtFormDesignMapper.xml（字段 form_structure_json） |
| 一个客户填了两次同一个表单，怎么查最新的 | ExtFormDataMapper.xml selectLatestByFormKeyAndBizId |
| 某角色看不到某字段 → 字段权限配置存哪 | FormFieldPermissionMapper 或 ModuleFieldRolePermissionMapper，核对实际用哪套 |

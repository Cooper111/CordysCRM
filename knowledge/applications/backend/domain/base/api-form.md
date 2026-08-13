---
type: domain-base
sub-type: api-index
module: form
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/form/controller 目录
owner: cordys-crm
verify-required: true
verify-note: 自定义表单的控件类型/校验规则 JSON/模板渲染 接口易变，核对代码
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/form.md
---

# 自定义表单 模块 API 索引（api-form）

---

## 一、Controller 类定位

| Controller 类名 | 职责一句话 | 对应 Service | 权限节点前缀 |
|---|---|---|---|
| **FormDesignController** | 表单设计器：保存 JSON 结构（字段/控件/栅格）、发布、版本号、预览、复制表单 | FormDesignService + FormVersionService | `FORM_DESIGN:ADD/EDIT/DELETE/PUBLISH` |
| **FormDataController** | 表单数据运行时：保存表单实例数据、按 form_key 查询、校验字段必填/校验规则、表单数据版本 | FormDataService | `FORM_DATA:ADD/EDIT/DELETE/VIEW` + 对应业务主档权限 |
| **FormPermissionController**（如有独立） | 表单角色/字段可见性/字段可编辑性 CRUD：按角色分配表单字段权限 | FormFieldPermissionService | `FORM_PERMISSION:MANAGE` |
| **FormTemplateController** | 表单模板：模板列表、模板复制、模板启用/停用、模板预览 | FormTemplateService | `FORM_TEMPLATE:MANAGE` |

> 注：Form 模块常与 system.ModuleField 协作（模块字段扩展的两种模式：一种 Field+FieldBlob 业务域自己管，一种 Form 引擎管）。若 Form 实际走 ModuleField，定位入口见 api-system.md 的 ModuleFieldController。

---

## 二、定位技巧

| 关键词 | Controller |
|---|---|
| 表单设计器/保存结构/发布 | FormDesignController |
| 填表单数据/保存数据/加载已填数据 | FormDataController |
| 表单字段权限（某角色不可见某字段） | FormPermissionController 或 system.ModuleField 内的权限配置 |
| 表单模板（预置模板） | FormTemplateController |

路径前缀（核对 @RequestMapping）：
```
/api/form-design/*     → 设计
/api/form-data/*       → 数据
/api/form-permission/* → 权限
/api/form-template/*   → 模板
```

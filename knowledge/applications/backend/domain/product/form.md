---
type: domain-product
app: backend
module: form
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/form 下 CustomForm / CustomFormData / CustomFormRole / CustomFormAdmin / CustomFormRoleUser / CustomFormDataField + 对应 Controller/Service/Mapper/Export
owner: cordys-crm
verify-required: true
verify-note: 表单角色类型、字段扩展 Blob 结构、数据导出规则需回代码核对
updated: 2026-08-11
related:
  - knowledge/main/glossary.md#自定义表单
  - knowledge/applications/backend/domain/base/api-form.md
  - knowledge/applications/backend/domain/base/repository-form.md
---

# 自定义表单模块 · 产品能力

---

## 一、能力概述

提供"非标准业务对象"的快速建模与数据管理能力：管理员通过**表单设计器**设计一张"自定义表单"（字段/布局/分组），普通用户在对应菜单下录入"表单数据"（支持字段扩展 Blob + 自定义字段权限 + 角色权限）。典型用于 CRM 无法预置的业务场景（如市场活动登记、供应商档案、项目工时...）。

两大部分：
1. **表单设计（CustomForm）**：设计态，管理一张表单的结构（字段、分组、布局、角色权限）
2. **表单数据（CustomFormData）**：运行态，一张表单的多条数据行（相当于一张自定义表）

主干流程：
```
管理员：表单设计（字段/分组/布局）→ 表单角色配置 → 分配表单给角色可见
普通用户：表单数据列表 → 新增 → 按设计好的字段填写 → 提交保存 → 查询/导出
```

---

## 二、核心流程（ASCII 流程图）

```
【设计态 CustomFormService】
    管理员新建表单（表单名+描述）
        │ 编辑字段（CustomForm 字段配置或复用 system.ModuleField？按实现核对）
        │ 编辑布局（分组/排序/必填/类型）
        ▼
    配置 CustomFormRole（表单角色：管理员/只读/填写/无权限）
        └ 表单角色绑定系统角色 + CustomFormRoleUser（额外指定用户）
        │
        ▼
    启用表单 → 前端菜单动态出现该表单
        │
        ▼
【运行态 CustomFormDataService】
    具有该表单"填写"权限的用户：
        → 新增表单数据 CustomFormData
            - CustomFormData 主表（meta：formId + ownerId + dataScope）
            - CustomFormDataField + CustomFormDataFieldBlob（字段扩展 Blob，存各字段值）
            - 操作日志 CustomFormDataLogService
        → 列表查询：按 DataScopeService + 表单角色权限过滤
        → 详情查看 / 编辑 / 删除
        → 导出数据（CustomFormDataExportService）：按权限裁剪列，大导出异步
```

### 步骤详解

| 步骤 | 触发条件 | 核心逻辑要点 | 代码入口 |
|---|---|---|---|
| 表单设计（新建/编辑） | 管理后台→自定义表单→新建 | CustomForm + 字段配置 + 启用/停用开关；保存后缓存表单定义失效，下次查询重建 | CustomFormService + CustomFormLogService |
| 表单角色配置 | 表单设计→角色 Tab | CustomFormRole（按系统角色配置类型：ADMIN / READONLY / WRITE / NONE）+ CustomFormRoleUser（例外：对特定用户单独授权）→ 两者合并为用户最终权限 | CustomFormRoleService + ExtCustomFormRoleUserMapper |
| 表单数据新增 | 用户→表单页→新增 | 权限校验（WRITE）→ 字段校验（必填/类型/唯一）→ 写 CustomFormData + DataField + DataFieldBlob → 操作日志 | CustomFormDataService.add |
| 表单数据列表查询 | 表单列表页 | CsPermission 节点（FORM:VIEW）→ 表单角色过滤 → DataScope（SELF/DEPT/ALL）→ 表单字段权限过滤 → 分页结果 | CustomFormDataService.page（CustomFormDataPageRequest） |
| 表单数据导出 | 列表→导出 | CustomFormDataExportService：按导出权限列 → 小量同步、大量异步（ExportTask）→ 同 CRM 其他模块导出模式（tech/excel-io.md） | CustomFormDataExportService |
| 表单管理员操作（数据转移/批量删除） | 表单 ADMIN 角色 | 数据转移：修改 dataId 的 ownerId；批量删除：按角色权限过滤后批量 DELETE；所有批量操作写操作日志 | CustomFormDataService（Admin 方法，如 transferOwner / batchDelete） |

---

## 三、核心业务规则

| 规则 ID | 规则描述 | 例外 |
|---|---|---|
| R1 | **字段 Blob 最大 1MB**：if (CustomFormDataFieldBlob JSON 序列化后 size > 1MB) then 拒绝保存 | 管理员可通过 Parameter.CUSTOM_FORM_BLOB_LIMIT 调大（最多 10MB） |
| R2 | **表单数据查询权限合并公式**：最终权限 = FormRole（按系统角色）∪ CustomFormRoleUser（按用户单独指定）→ **最大权限优先**（如有 WRITE 则 WRITE，有 READ 则 READ） | 系统管理员 = 全表单 ADMIN 最高权限 |
| R3 | **删除表单=级联删除数据？否**：if (删除 CustomForm) → 逻辑删除 CustomForm → CustomFormData 保留（关联外键保留，前端不展示），提供"永久清理表单数据"按钮（仅超管） | 超管永久清理前提示：是否备份导出（推荐一键导出为归档 Excel） |
| R4 | **表单名唯一**：同组织 if (form.name 重复) then 拒绝编辑/新建 | — |
| R5 | **字段类型**：表单字段类型（文本/数字/日期/单选/多选/用户/部门/级联/附件）受 system.ModuleField 的 FieldType 统一约束；新增类型必须两边同步实现 | — |
| R6 | **导出字段权限**：导出时严格按"表单角色对该字段的查看权限"裁剪列，即使导出人是 owner | 超管可勾选"以管理员权限导出全字段"（操作日志记风险原因） |

---

## 四、状态/枚举（定位入口）

| 语义 | 枚举/类名入口 | 代码路径 |
|---|---|---|
| 表单状态（启用/停用/草稿） | CustomForm.status 字段 | crm.form.domain.CustomForm |
| 表单角色类型（管理员/只读/填写/无权限） | CustomFormRole.type 字段 + 枚举常量 | crm.form.domain.CustomFormRole + CustomFormRoleUser.type |
| 表单 Key（CustomForm 不通过 FormKey，使用 formId 做动态路由） | CustomForm.id 唯一标识每个业务表单；动态权限拼接 `FORM:{formId}:*` | crm.form.domain.CustomForm.id |

---

## 五、与其他模块的协作关系

| 协作模块 | 调用方向 | 关键契约 |
|---|---|---|
| **system（模块字段 / 字段类型）** | form → system | 表单字段类型定义与 system.ModuleField.FieldType 保持一致；字段渲染/取值复用 ModuleFieldService |
| **system（权限 CsPermission + DataScope）** | form → system | 表单操作统一走 CsPermission（FORM:ADD/EDIT/DELETE/EXPORT/ADMIN）；数据范围受 DataScopeService 过滤（resourceType=FORM_DATA + formId + dataId） |
| **system（操作日志 + 通知 + 导入导出）** | form → system | CustomFormLogService / CustomFormDataLogService（BaseModuleLogService 扩展）；大导出异步走 ExportTask；通知（分配表单管理员时） |
| **approval（审批，可选）** | form → approval | 如果组织配置"自定义表单数据提交需审批"→ CustomFormData 提交可触发 @HitApproval(FormType=CUSTOM_FORM + formId) |

---

## 六、常见边界场景 & 处理方式

| 场景 | 处理方式 | 代码入口 |
|---|---|---|
| **自定义表单数据 10W+ 条（大表）** | 自定义表单 Mapper 查询分页 + 索引：custom_form_data(formId, ownerId) 组合索引；大列表查询走条件过滤；Dashboard/报表建议同步到 DataEase | ExtCustomFormDataMapper + 对应 XML SQL 走分页 |
| **表单字段变化后老数据显示异常** | 老数据的 DataField 中已不存在的字段 → 前端展示为"（已废弃字段）"+ 值（保留可看）；不允许编辑这些字段（编辑页不渲染） | CustomFormDataFieldService 合并字段差异 + 前端标记废弃 |
| **角色变动后表单权限不同步** | 权限不缓存（每次实时计算：FormRole ∪ RoleUser），用户角色变更后下次查询立刻生效 | CustomFormRoleService.resolveUserFormPermission（实时查询） |
| **删除字段后老记录里的 Blob 仍有该字段（脏数据）** | Blob 不立即清理（保留审计）→ 下次 UPDATE 该数据时自动过滤掉已被删除的字段再序列化写回；定时离线清理 CleanTempResourceListener 可清理超过 1 年的脏 Blob | CustomFormDataFieldService.save 前 stripDeletedFields + CleanTempResourceListener |

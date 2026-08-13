---
type: domain-base
sub-type: domain-index
module: system
app: backend
status: official
evidence: backend/crm/src/main/java/cn/cordys/crm/system/ 目录 + framework 层 CsPermission、ModuleLogServiceFactory 等
owner: cordys-crm
verify-required: true
verify-note: 权限节点编码/模块字段扩展类型字典/日志表结构 易变
updated: 2026-08-11
related:
  - knowledge/applications/backend/domain/product/system.md
  - knowledge/applications/backend/domain/base/api-system.md
---

# 系统管理 模块 领域模型索引（domain-system）

---

## 一、核心 Domain 类定位

### 1. 权限/用户/角色/部门

| Domain 类名 | 职责一句话 | 表前缀 | ExtMapper | 字段扩展 |
|---|---|---|---|---|
| **SysUser** | 系统用户：账号、姓名、手机号、邮箱、密码(加密)、状态、部门 id、岗位、创建人、最后登录时间 | sys_user | SysUserExtMapper | N |
| **SysRole** | 角色：角色编码、角色名称、角色类型(系统/业务)、是否启用、数据范围（全部/本部门/本部门及下级/本人/自定义） | sys_role | SysRoleExtMapper | N |
| **SysDept**（或 Department） | 部门树：父 id、部门名称、负责人 userId、排序、状态 | sys_dept | SysDeptExtMapper | N |
| **UserRole** | 用户-角色 关联表（多对多）：userId、roleId | sys_user_role | UserRoleExtMapper | N |
| **RolePermission** | 角色-权限节点 关联表：roleId、permissionCode（如 CLUE:ADD） | sys_role_permission | RolePermissionExtMapper | N |
| **Permission** | 权限节点元数据表：permissionCode（唯一）、permissionName、所属模块(moduleKey)、parentCode(父节点)、节点类型（目录/菜单/按钮/数据权限）、注解来源类名+方法名（@CsPermission 扫描产物） | sys_permission | PermissionExtMapper | N |
| **DataScope**（若独立） | 数据范围自定义明细：roleId、deptId 集合（当 dataScope=CUSTOM 时，该角色能看哪些部门） | sys_role_data_scope_dept（核对） | DataScopeExtMapper | N |

### 2. 模块字段扩展（通用能力）

| Domain 类名 | 职责一句话 | 表前缀 | ExtMapper | 字段扩展 |
|---|---|---|---|---|
| **ModuleField** | 模块字段元数据：moduleKey（CLUE/CUSTOMER/OPPORTUNITY 等）、fieldKey、fieldName、fieldType（text/textarea/number/date/datetime/select/multiSelect/radio/checkbox/user/dept/file/image/cascader 等）、是否必填、默认值、排序、分组 key、字典分类 dictCategoryKey、是否系统字段(不可删)、是否可见/可编辑的角色权限 JSON | sys_module_field | ModuleFieldExtMapper | N |
| **ModuleFieldGroup** | 模块字段分组：moduleKey、groupKey、groupName、排序 | sys_module_field_group | ModuleFieldGroupExtMapper | N |
| **ModuleFieldRolePermission**（若独立） | 字段-角色可见/可编辑 权限：fieldId、roleId、visible(YN)、editable(YN) | sys_module_field_role_permission | ModuleFieldRolePermissionExtMapper | N |

### 3. 操作日志

| Domain 类名 | 职责一句话 | 表前缀 | ExtMapper | 字段扩展 |
|---|---|---|---|---|
| **OperationLog** | 操作日志主档：logId、moduleKey、bizId、operationType（ADD/EDIT/DELETE/APPROVE/EXPORT/IMPORT…）、操作人 userId、操作人名称、操作时间、操作人 IP、请求 URL、请求参数 JSON、返回状态码、异常堆栈（若失败）、变更前后 diff JSON | sys_operation_log（或按月份分表 sys_operation_log_202608，核对） | OperationLogExtMapper | N |
| **OperationLogDiff**（若独立表） | 字段级 diff：logId、fieldKey、oldValue、newValue | sys_operation_log_diff（核对） | OperationLogDiffExtMapper | N |

### 4. 系统配置 / 字典 / 筛选视图

| Domain 类名 | 职责一句话 | 表前缀 | ExtMapper | 字段扩展 |
|---|---|---|---|---|
| **SystemConfig** | 系统参数：configKey、configValue、valueType(string/number/boolean/json)、描述、所属分组、是否需要重启生效 | sys_system_config | SystemConfigExtMapper | N |
| **Dictionary** | 字典分类表：dictCategoryKey、dictCategoryName、是否启用、排序、描述 | sys_dictionary（或 sys_dict_category） | DictionaryExtMapper | N |
| **DictionaryItem** | 字典项：dictCategoryKey、dictItemKey、dictItemValue、排序、是否启用、父 item key（级联字典）、扩展属性 JSON | sys_dictionary_item | DictionaryItemExtMapper | N |
| **UserView** | 用户筛选视图：moduleKey、userId、viewName、viewScope（PRIVATE/PUBLIC）、filterCondition JSON（查询条件结构）、sortCondition JSON、是否默认视图 | sys_user_view | UserViewExtMapper | N |

### DTO 包位置
- request/response：`cn.cordys.crm.system.dto.*`
- 权限注解类：`cn.cordys.framework.security.annotation.CsPermission`（framework 层）

---

## 二、定位技巧

| 需求关键词 | 找哪个类 |
|---|---|
| 新增用户 / 重置密码 / 用户列表 | SysUser |
| 角色有哪些 / 给角色加权限 | SysRole + RolePermission |
| 部门树 / 部门负责人 | SysDept |
| "为什么我没权限" / 权限检查失败 / 权限树刷新 | Permission 表 + CsPermission 注解扫描逻辑（framework） |
| 某字段在列表里加了没显示 / 字段扩展 / 自定义字段 | ModuleField + ModuleFieldGroup + ModuleFieldRolePermission |
| 谁改了某条线索的负责人 / 操作日志 / 变更 diff | OperationLog + OperationLogDiff（若独立） |
| 系统参数 / "线索回收超时天数"在哪里改 | SystemConfig |
| 下拉选项枚举 / 字典项 / 级联字典 | Dictionary + DictionaryItem |
| 列表筛选条件 保存为视图 | UserView |

---

## 三、枚举入口

| 枚举类名 | 说明 |
|---|---|
| **UserStatusEnum** | 用户状态（正常/停用/锁定/离职…） |
| **DataScopeEnum** | 数据范围枚举（ALL/DEPT/DEPT_AND_SUB/SELF/CUSTOM） |
| **ModuleFieldTypeEnum** | 字段类型枚举（text/number/date/select/…，与前端控件一一对应） |
| **OperationTypeEnum** | 操作类型枚举（ADD/EDIT/DELETE/IMPORT/EXPORT/APPROVE…，ModuleLogServiceFactory 常用） |
| **PermissionNodeTypeEnum** | 权限节点类型（目录/菜单/按钮/数据） |
| **ViewScopeEnum** | 视图范围（PRIVATE/PUBLIC） |
